> **CONFIDENTIAL — AUTHORISED PERSONNEL ONLY**
> This repository contains a training-program deliverable. Do not make this repository public — see [Handling & disclaimer](#handling--disclaimer) below.

# Penetration Testing Report

**Black-box Penetration Test & Vulnerability Assessment**

`W4-PM-FINAL` · Cybersecurity · Networkwalks

| | |
|---|---|
| **Pentester** | Muhammad Saad Rizwan |
| **Program / Batch** | B084 — Networkwalks |
| **Report date** | 23 November 2026 |
| **Modules completed** | M1 Initial Access \| M2 Data Extraction \| M3 Data Exposure \| M4 Reporting |
| **Client / target** | Mediroza General Hospital — https://medirozahospital.com |
| **Authorisation** | Written authorisation for the target domain only |
| **Assessment** | Black-box penetration test and vulnerability assessment. Engagement duration: 5 days, as defined in the project brief |

**OVERALL RISK: CRITICAL**

> Evidence indicates access to patient-report content, exposed directory indexes, and an internal HR/shareholder database backup. Ratings are qualitative and based only on supplied evidence.

`W4-PM-FINAL` • Week 04 Internship Project

## Contents

1. [Liability Disclaimer](#1-liability-disclaimer)
2. [Introduction](#2-introduction)
3. [Tools Used](#3-tools-used)
4. [Activities Performed](#4-activities-performed)
5. [Risk Analysis / Impact](#5-risk-analysis--impact)
6. [Recommendations](#6-recommendations)
7. [Conclusion](#7-conclusion)
8. [Evidence Collected](#8-evidence-collected)
9. [Handling & Disclaimer](#handling--disclaimer)

Overall risk is rated Critical based on the supplied evidence. The report distinguishes observed evidence from unverified access paths and clearly records the limits of the evidence package.

## 1. Liability Disclaimer

I performed these activities only on systems for which written authorisation was granted. The project brief limited testing to the target domain and prohibited social engineering, denial-of-service testing, and testing outside the agreed scope. All materials are for educational and research purposes only. Do not use anything in this report to break the law. The instructor, authors, and Networkwalks are not responsible for the use of this knowledge; every action taken is the user's own responsibility. Misuse can lead to criminal charges, heavy fines, loss of employment, and a permanent record. In most countries, unauthorised access is a crime even when nothing is damaged.

## 2. Introduction

This report documents a black-box assessment of the Mediroza General Hospital web application (https://medirozahospital.com), completed as Week 4 of the author's ongoing internship program at Networkwalks. The supplied evidence indicates access to a patient-report portal, directory indexes under patient and staff paths, and a database backup containing human-resources and shareholder records. The material indicates a serious confidentiality risk because exposed information includes patient laboratory reports and staff payroll and identity data.

The strongest evidence is a database-dump screenshot identifying an internal HR database and showing staff and shareholder table structures and records. The patient portal screenshot lists three password-protected pathology reports, and separate screenshots show report contents after opening. Directory-index screenshots show the server exposing file and directory names under `/patient/` and `/staff/`.

Overall risk is rated Critical, based on the apparent exposure of patient health information and confidential HR/shareholder data. This is a qualitative assessment of the supplied evidence, not a verified production-impact measurement.

**Evidence qualification:** the screenshots do not include a complete exploit transcript for initial access, a final password-cracking result, or the exact URL/path from which the SQL backup was retrieved. Those gaps are recorded below and should be closed before treating the assessment as fully reproducible.

### Key outcomes by module

| Module | Evidence-based outcome | Risk / status |
|---|---|---|
| M1 — Initial access | Patient portal page displays three encrypted pathology reports; report-content screenshots are supplied. | Critical |
| M2 — Data extraction | PDF hash extraction and cracking attempts are shown. Supplied terminal evidence does not show a confirmed recovered-password result. | Unconfirmed |
| M3 — Data exposure | Directory indexes and a database dump containing staff/payroll and shareholder records are shown. | Critical |
| M4 — Reporting | This report consolidates findings, evidence references, risk ratings, and remediation actions. | Complete |

## 3. Tools Used

The following tools are visible in the supplied evidence. No additional tools are asserted because they are not evidenced in the supplied material.

| Tool | Purpose |
|---|---|
| Kali Linux | Operating system and terminal used for the extraction and cracking activities. |
| Web browser | Review of the public website, patient portal, and `/patient/` and `/staff/` directory indexes. |
| Manual SQL injection | An authentication-bypass payload (`admin' -- -`) submitted in the patient portal login form's username field. |
| John the Ripper | Attempted recovery of the PDF passwords from extracted hashes; output reported "No password hashes loaded". |
| Hashcat v7.1.2 | Attempted PDF password recovery using mode 10500 with a wordlist attack (`-a 0`). |
| RockYou wordlist | Wordlist (`/usr/share/wordlists/rockyou.txt`) used for the password-recovery attempts. |

## 4. Activities Performed

### 4.1 Scope and Methodology

**Target:** https://medirozahospital.com · **Client:** Mediroza General Hospital · **Assessment:** full black-box penetration test and vulnerability assessment, as described in the Week 4 brief. Testing was defined as a five-day independent exercise, with findings not to be discussed with other participants before the reveal session.

The supplied evidence indicates the following activities:
1. Review of the public website and patient portal.
2. Submission of a SQL-injection authentication-bypass payload against the patient portal login form.
3. Retrieval of three encrypted PDF pathology reports.
4. Extraction of PDF password hashes.
5. Attempted password recovery using John the Ripper and Hashcat with a wordlist.
6. Review of exposed directory indexes.
7. Inspection of an SQL backup containing staff and shareholder tables.

### 4.2 Findings and Proof of Exploitation

#### F-01 — Sensitive patient reports accessible through patient portal

**Severity:** CRITICAL · **Evidence confidence:** High for report access; moderate for the authentication-bypass path

**Description:** The patient portal login screenshot shows a SQL-injection authentication-bypass payload (`admin' -- -`) submitted in the username field. The patient portal screenshot displays three password-protected pathology report download entries, and separate screenshots show three pathology laboratory report pages opened and readable. This demonstrates access to patient-report content in the supplied evidence. The screenshots do not capture the login response following the payload submission, so they do not by themselves confirm that this payload was the successful access path; they confirm only that it was attempted against the login form.

**Impact:** Unauthorised access to laboratory reports can expose protected health information and may enable privacy violations, targeted fraud, reputational damage, and regulatory consequences. A successful authentication-bypass vulnerability in the login form would also indicate unsanitised input reaching the backend query, a serious application-layer flaw.

**Evidence:** [Evidence 1](evidence/01_patient_portal_sqli_payload.jpg) shows the SQL-injection payload submitted at the portal login; [Evidence 2](evidence/02_patient_portal_report_list.jpg) shows the portal report list; [Evidence 3](evidence/03_opened_report_redacted_a.jpg)–[4](evidence/04_opened_report_redacted_b.jpg) show opened report pages. Patient-identifying and clinical details are redacted in the appendix.

**Recommendation:** Enforce server-side authorisation on every report request; bind each report to the authenticated patient; use opaque, short-lived download tokens; prevent direct object references; log and alert on cross-account access; use parameterised queries/prepared statements on every database-backed form and validate input server-side to remediate SQL injection; review access logs and notify appropriate privacy/security stakeholders as required.

#### F-02 — Directory listing enabled on patient and staff paths

**Severity:** HIGH · **Evidence confidence:** High

**Description:** The supplied browser screenshots show directory index pages for `/patient/` and `/staff/`. The patient index exposes filenames including reports, `download.php`, `error_log`, `login.php`, `logout.php`, and `portal.php`. The staff index exposes a `login.php` entry. The screenshots identify LiteSpeed on port 443.

**Impact:** Directory indexes disclose application structure and potentially sensitive operational files. If backups, logs, source files, or reports are present in these directories, an attacker may discover and retrieve them.

**Evidence:** [Evidence 5](evidence/05_patient_directory_index.jpg) shows the `/patient/` index; [Evidence 6](evidence/06_staff_directory_index.jpg) shows the `/staff/` index.

**Recommendation:** Disable directory indexing at the web-server and virtual-host levels; deny direct web access to logs, backups, and application internals; move sensitive files outside the document root; enforce authentication and authorisation on every endpoint; verify the deployed configuration after remediation.

#### F-03 — Exposure of internal HR database backup

**Severity:** CRITICAL · **Evidence confidence:** High for sensitive backup contents; moderate for the exact exposure mechanism

**Description:** A screenshot of `mediroza_db_backup_2019.sql` identifies an internal Mediroza General Hospital database backup for the `mediroza_hr` database. It contains staff and shareholder tables. The visible staff schema includes full name, job title, department, email, phone, national ID, monthly salary in ZAR, and date joined. Five staff records are visible in the supplied screenshot; the screenshot is clipped and does not establish the full employee count.

**Impact:** The backup combines employee identity data, contact details, and compensation information. Exposure increases risks of identity theft, targeted phishing, extortion, and internal privacy harm. The backup also reveals internal database structure.

**Evidence:** [Evidence 7](evidence/07_hr_database_backup.jpg) shows the backup header, schema, and a partial set of staff rows. Direct identifiers and row-level details are redacted in the appendix. The screenshot does not show the exact URL or retrieval request used to obtain the backup.

**Recommendation:** Remove database backups from public web roots; rotate any credentials or secrets that may have been included; restrict backup storage using least-privilege access controls and encryption; implement backup inventory and retention policies; scan web roots for `.sql`, `.zip`, `.bak`, and dump files; review access logs for possible download activity.

#### F-04 — Confidential shareholder ownership information exposed

**Severity:** HIGH · **Evidence confidence:** High

**Description:** The database-dump screenshot contains a shareholders table with ten entries. The visible ownership percentages are 18%, 15%, 12%, 11%, 10%, 9%, 8%, 7%, 6%, and 4%, totalling 100%. The table also records share counts and share classes. Individual names are intentionally omitted from this report.

**Impact:** Disclosure of ownership structure can expose confidential corporate information and support targeted social engineering, business-intelligence gathering, or attempts to impersonate stakeholders.

**Evidence:** [Evidence 8](evidence/08_shareholder_table.jpg) shows the shareholder table schema and all ten ownership entries. Names are redacted in the appendix.

**Recommendation:** Treat shareholder records as confidential business data; restrict database access to authorised finance/legal roles; remove backups from public paths; encrypt backups at rest and in transit; monitor access and establish a controlled disclosure process.

#### F-05 — PDF password-recovery evidence is incomplete

**Severity:** INFORMATIONAL / VALIDATION REQUIRED · **Evidence confidence:** High confidence in the incomplete evidence record

**Description:** The terminal screenshots show PDF hash material being collected, an attempted John the Ripper command that reports "No password hashes loaded", and a Hashcat v7.1.2 run initiated with mode 10500 and the RockYou wordlist. The captured Hashcat output shows startup and hash/device information but no final status or recovered passwords. Therefore, the screenshots do not independently prove successful password cracking. The opened report screenshots establish readable report content but do not prove the exact password-recovery method.

**Impact:** Without a complete result log, the assessor cannot verify which passwords were recovered, which tool/mode succeeded, or whether all three files were successfully decrypted through the documented process.

**Evidence:** [Evidence 9](evidence/09_pdf_hash_extraction.jpg) shows extracted hash material and wordlist preparation; [Evidence 10](evidence/10_john_the_ripper_attempt.jpg) shows the unsuccessful John load attempt; [Evidence 11](evidence/11_hashcat_startup.jpg) shows Hashcat startup; Evidence 3–4 show readable reports.

**Recommendation:** Preserve the original encrypted PDFs and hashes; capture the final cracking output showing recovered status and the matching hash-to-file mapping; document tool version, exact command, wordlist, and timestamps; retain decrypted copies securely and redact patient data in any report evidence.

### 4.3 Limitations and Data Handling

The source ZIP contains screenshots rather than original PDF reports, raw HTTP requests/responses, a full terminal session, or the SQL backup file itself. Therefore, this report cannot independently validate file hashes, password recovery, server response headers, the exact backup URL, the complete number of staff records, or whether the screenshots represent a live production environment. The backup header shows a 2019-08-27 timestamp; its age does not reduce the sensitivity of the records shown.

Patient names, lab identifiers, dates of birth, clinical values, staff contact details, and national identification numbers are not reproduced in this report. Evidence images have been redacted where practical. This report summarises the nature and impact of the exposure without republishing sensitive records.

## 5. Risk Analysis / Impact

Based on the evidence reviewed, the following risks were identified:

| # | Risk / finding | Evidence / observation | Potential impact | Risk level |
|---|---|---|---|---|
| 1 | F-01 Patient report access | Patient laboratory report content is visible in supplied evidence. | Exposure of protected health information; privacy and regulatory consequences. | Critical |
| 2 | F-02 Directory listing | Internal paths and filenames are disclosed. | Application structure disclosed; sensitive files may be discovered. | High |
| 3 | F-03 HR backup exposure | Payroll and employee identity fields appear in a database dump. | Identity theft, phishing, extortion, and internal privacy harm. | Critical |
| 4 | F-04 Shareholder data | Confidential ownership and share-class information is exposed. | Corporate intelligence gathering; stakeholder impersonation. | High |
| 5 | F-05 PDF cracking proof gap | Final recovery result is not present in supplied terminal screenshots. | Milestone cannot be reliably confirmed. | Validation required |

**Risk-level key:** Critical · High · Validation required.

**Severity definitions:**
- **Critical:** direct exposure of highly sensitive patient or employee/corporate data with potentially severe confidentiality impact.
- **High:** significant disclosure of internal application structure or confidential business information.
- **Validation required:** evidence gap that prevents a reliable conclusion about completion of a milestone.

These ratings are a qualitative assessment of the supplied evidence, not a verified production-impact measurement.

## 6. Recommendations

Based on the findings, the following security improvements are recommended, in priority order:

### Priority 1 — Immediate containment

1. **Remove exposed files.** Remove all database dumps, archives, logs, and report files from publicly reachable web directories.
2. **Disable directory indexing.** Disable directory indexing on the web server and confirm that `/patient/` and `/staff/` no longer return directory listings.
3. **Review logs.** Review web, application, and server logs for access to the backup, patient reports, and sensitive endpoints.
4. **Restrict report downloads.** Restrict patient report downloads to the correct authenticated patient and verify authorisation on the server for every request.
5. **Escalate to leadership.** Engage the organisation's privacy/security leadership to assess whether exposed patient or employee data requires incident response or notification.

### Priority 2 — Access control and data protection

1. **Secure backup storage.** Store backups outside the document root in access-controlled storage.
2. **Encrypt backups.** Encrypt backups using managed keys and restrict decryption permissions.
3. **Apply least privilege.** Apply least privilege to HR and shareholder database tables; separate application accounts from administrative accounts.
4. **Harden download links.** Use unpredictable, time-limited download tokens and prevent insecure direct object references.
5. **Clean error logs.** Remove sensitive data from application error logs and prevent public access to operational files.

### Priority 3 — Secure configuration and assurance

1. **Automate deployment checks.** Add automated deployment checks that fail builds when directory listing is enabled or backup files exist in public paths.
2. **Test regularly.** Conduct periodic external exposure scans and authenticated access-control tests.
3. **Monitor access.** Implement monitoring for unusual report downloads, bulk access, and access to backup extensions.
4. **Govern backups.** Establish backup retention, secure deletion, and access-review procedures.
5. **Retest.** Retest all findings after remediation and preserve before/after evidence.

### Retest acceptance criteria

The retest should confirm that:
- (a) directory requests return 403/404 or an appropriate controlled response rather than a listing;
- (b) backup files cannot be fetched from the public internet;
- (c) unauthorised users cannot view or download another patient's reports;
- (d) HR and shareholder data are inaccessible to unprivileged accounts; and
- (e) security logs capture denied and suspicious access attempts.

## 7. Conclusion

The supplied evidence indicates material weaknesses in information-exposure controls around the Mediroza General Hospital web application. The most serious issues are the presence of patient-report content in the assessment evidence and an internal HR database backup containing employee and shareholder information. Directory indexing further reveals application paths and filenames that should not be publicly enumerated.

The organisation should prioritise containment of exposed files, enforcement of object-level authorisation for patient reports, and review of access logs. The PDF cracking milestone should be supplemented with complete final tool output before claiming that the password-recovery process has been conclusively demonstrated for all three files.

This report is based solely on the 13 screenshots in `W4.zip` and the Week 4 project brief. It is not an independent validation of the live target or a substitute for a complete evidence package containing original files, raw requests/responses, and full command output.

## 8. Evidence Collected

### Evidence register

| File | What it supports |
|---|---|
| [01_patient_portal_sqli_payload.jpg](evidence/01_patient_portal_sqli_payload.jpg) | Patient portal login page, showing a SQL-injection authentication-bypass payload submitted in the username field. |
| [02_patient_portal_report_list.jpg](evidence/02_patient_portal_report_list.jpg) | Patient portal listing three encrypted pathology reports; patient identifiers redacted. |
| [03_opened_report_redacted_a.jpg](evidence/03_opened_report_redacted_a.jpg) | Opened pathology report screenshot; patient and clinical details redacted. |
| [04_opened_report_redacted_b.jpg](evidence/04_opened_report_redacted_b.jpg) | Opened pathology report screenshot; patient and clinical details redacted. |
| [05_patient_directory_index.jpg](evidence/05_patient_directory_index.jpg) | Directory index for `/patient/`. |
| [06_staff_directory_index.jpg](evidence/06_staff_directory_index.jpg) | Directory index for `/staff/`. |
| [07_hr_database_backup.jpg](evidence/07_hr_database_backup.jpg) | HR database backup schema and partial staff records; row-level identifiers redacted. |
| [08_shareholder_table.jpg](evidence/08_shareholder_table.jpg) | Shareholders table with ten ownership percentages; names redacted. |
| [09_pdf_hash_extraction.jpg](evidence/09_pdf_hash_extraction.jpg) | PDF hash material and wordlist preparation in Kali terminal. |
| [10_john_the_ripper_attempt.jpg](evidence/10_john_the_ripper_attempt.jpg) | John the Ripper attempt; terminal reports no hashes loaded. |
| [11_hashcat_startup.jpg](evidence/11_hashcat_startup.jpg) | Hashcat v7.1.2 startup and hash/device details; no final recovery status shown. |

The figures below reproduce the already-redacted screenshots included in the supplied report. They are presented as evidence references, not as independent validation of the live target.

![Figure 1. Patient portal login form with a SQL-injection authentication-bypass payload submitted in the username field; the resulting login response was not captured.](evidence/01_patient_portal_sqli_payload.jpg)
*Figure 1. Patient portal login form with a SQL-injection authentication-bypass payload (`admin' -- -`) submitted in the username field; the resulting login response was not captured.*

![Figure 2. Patient portal report list; patient names and reference identifiers are redacted.](evidence/02_patient_portal_report_list.jpg)
*Figure 2. Patient portal report list; patient names and reference identifiers are redacted.*

![Figure 3. Opened pathology report; patient details redacted.](evidence/03_opened_report_redacted_a.jpg)
*Figure 3. Opened pathology report; patient name, ID, date of birth, referring doctor, and all clinical results are redacted.*

![Figure 4. Opened pathology report; patient details redacted.](evidence/04_opened_report_redacted_b.jpg)
*Figure 4. Opened pathology report; patient name, ID, date of birth, referring doctor, and all clinical results are redacted.*

![Figure 5. /patient/ directory index.](evidence/05_patient_directory_index.jpg)
*Figure 5. `/patient/` directory index.*

![Figure 6. /staff/ directory index.](evidence/06_staff_directory_index.jpg)
*Figure 6. `/staff/` directory index.*

![Figure 7. Internal HR database backup; record-level details redacted.](evidence/07_hr_database_backup.jpg)
*Figure 7. Internal HR database backup; record-level details are redacted.*

![Figure 8. Shareholder table; names redacted.](evidence/08_shareholder_table.jpg)
*Figure 8. Shareholder table; names are redacted, while ownership percentages and share classes are retained.*

![Figure 9. PDF hash extraction and wordlist preparation.](evidence/09_pdf_hash_extraction.jpg)
*Figure 9. PDF hash extraction and wordlist preparation.*

![Figure 10. John the Ripper attempt and "No password hashes loaded" output.](evidence/10_john_the_ripper_attempt.jpg)
*Figure 10. John the Ripper attempt and "No password hashes loaded" output.*

![Figure 11. Hashcat startup output; no final recovered status is shown.](evidence/11_hashcat_startup.jpg)
*Figure 11. Hashcat startup output; no final recovered status is shown.*

## Report Author and Project Information

- **Pentester:** Muhammad Saad Rizwan
- **Program / Batch:** B084 — Networkwalks
- **Report:** W4-PM-FINAL, Week 04 Internship Project
- **Report date:** 23 November 2026

---

## Handling & Disclaimer

This repository is a **training-program deliverable**, produced as part of a scoped, written-authorisation internship exercise against a designated practice target. It is marked confidential and is intended for authorised personnel only (instructor/program review).

- **Keep this repository private.** It names a specific target and documents findings (an authentication-bypass payload, exposed file/directory paths, and confirmation of an exposed HR/shareholder backup) that, per the report itself, had not been confirmed as remediated at time of writing. Evidence images have already been redacted by the author (see `evidence/`), but the descriptive findings are still enough to fail this on a public host.
- All testing described was performed only within the written authorisation and scope defined by the project brief. Do not use any technique, payload, or path described here against any system you are not explicitly authorised to test.
- If this relates to a real organisation and the issues described are not yet fixed, follow responsible-disclosure practice: report directly and privately to the organisation (or its designated security contact) rather than redistributing this report.
