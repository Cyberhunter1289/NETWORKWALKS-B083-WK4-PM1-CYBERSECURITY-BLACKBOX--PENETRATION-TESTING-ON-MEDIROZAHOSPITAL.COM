# NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX-PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM
This is the Networkwalks B083 Week 4 Blackbox Penetration Testing on Medirozahospital.com with a written permission and scope

**Black-Box Web Application Penetration Test & Vulnerability Assessment**

| | |
|---|---|
| **Target** | `https://medirozahospital.com` |
| **Client** | Mediroza General Hospital |
| **Assessment Type** | Full Black-box Web Application Penetration Test |
| **Assessment Period** | 27 September 2026 – 01 October 2026 |
| **Duration** | 5 days |
| **Testing Environment** | Kali Linux (VirtualBox) |
| **Authorization** | Written Authorization provided by the client |
| **Excluded Activities** | Social Engineering, Denial-of-Service, Out-of-Scope Testing |
| **Prepared By** | OPEYEMI AROWOSAFE |
| **Classification** | CONFIDENTIAL |

> This document summarizes a confidential assessment report. Evidence containing patient, staff, or shareholder data that has been exposed.

## NetworkWalks

```
N E T W O R K W A L K S
```

**Penetration Testing Project — Mediroza General Hospital**
Batch B083 | Week 4
Target: `https://medirozahospital.com`

| | |
|---|---|
| **Client** | Mediroza General Hospital |
| **Type** | Black-box Pentest |
| **Duration** | 5 Days |

> This project was conducted in a controlled environment for educational purposes only. These techniques must never be applied to any system without explicit written permission from the owner.

## Table of Contents

- [Purpose](#-purpose)
- [Executive Summary](#-executive-summary)
- [Scope & Rules of Engagement](#-scope--rules-of-engagement)
- [Methodology](#-methodology)
- [Tools & Techniques](#-tools--techniques)
- [Reconnaissance](#-reconnaissance)
- [Findings](#-findings)
- [Overall Risk Summary](#-overall-risk-summary)
- [Recommendations](#-recommendations)
- [Evidence Handling & Redaction](#-evidence-handling--redaction)
- [Conclusion](#-conclusion)
- [Disclaimer](#-disclaimer)

---

## Purpose

The purpose of this engagement was to help improve the security of the Mediroza General Hospital Web Application under an Authorized black-box test, identify security weaknesses, and demonstrate their practical impact through controlled exploitation.

The assessment addressed three trainer-defined milestones:

1. Attack the website and locate the 3 confidential PDF lab reports of patients.
2. Crack the encryption on all 3 retrieved files.
3. Find the critical data exposure on the client server.

---

## Executive Summary

A five-day black-box penetration test was conducted against the Mediroza General Hospital web application. Testing identified significant weaknesses affecting **authentication**, **protection of confidential patient reports**, and **exposure of internal organizational data**.

The most significant finding was a **SQL injection vulnerability** in the patient portal authentication mechanism. A MySQL syntax error confirmed that user-controlled input reached backend SQL processing, and further testing demonstrated a full **authentication bypass**. This exposed three password-protected pathology laboratory reports, whose passwords were subsequently recovered via **Networkwalks hash calculator and password cracker** 

A separate, unrelated critical exposure was identified through `robots.txt`, which disclosed a legacy `/old/` path. That path served a public directory listing containing an **unauthenticated internal SQL database backup** with confidential staff and shareholder records.

**Overall Risk: CRITICAL** — two Critical findings and one High finding were demonstrated.

| ID | Finding | Severity | Primary Impact |
|---|---|---|---|
| F-01 | SQL Injection -> Patient Portal Authentication Bypass | Critical | Unauthorized access to patient reports |
| F-02 | Weak Password Protection on Patient Reports | High | Recovery of passwords protecting sensitive reports |
| F-03 | Unauthenticated Exposure of Internal Database Backup | Critical | Exposure of staff/shareholder records |

**Sensitive data categories demonstrated as exposed:**

- Patient personally identifiable information (PII)
- Patient laboratory / health information
- Staff personal and employment information
- Staff contact and national identification information
- Staff salary information
- Shareholder and ownership information

---

## Scope & Rules of Engagement

**In scope:**
```
https://medirozahospital.com
```

**Rules of Engagement:**
- Testing limited strictly to the target domain
- Social engineering excluded
- Denial-of-service testing excluded
- Testing outside the agreed scope prohibited
- Conducted under written client authorization

**Environment covered:** public site, patient-facing functionality, staff authentication functionality, and legacy web resources.

---

## Methodology

Testing followed the three trainer-supplied milestones, performed manually via browser and Linux CLI tooling, with controlled exploitation limited to the authorized target.

```
Phase 1  Reconnaissance         -> whois.domaintools, whatweb, wafw00f, nmap, curl -i, robots.txt review
Phase 2  Authentication Testing -> SQL injection probing on patient login
Phase 3  Exploitation            -> auth bypass -> patient portal access
Phase 4  Data Extraction         -> 3 encrypted lab report PDFs retrieved
Phase 5  Offline Cracking        -> hash calculator, password checker -> all 3 recovered
Phase 6  Further Exposure Check -> robots.txt -> /old/ -> exposed DB backup
```

---

## Tools & Techniques

| Tool / Technique | Purpose |
|---|---|
| whois.domaintools |Information Gathering about DNS and Databases |
| whatweb | Identify the Technologies running on a Server |
| wafw00f | Check if there's a Firewall Protecting the packets |
| Nmap | Mapping Out Entire Network, Hosts and Infrastructure |
| ICMP / ping | Basic host connectivity testing |
| cURL | HTTP response and application reconnaissance |
| Web browser | Manual web application testing |
| SQL injection testing | Authentication and input validation testing |
| Hash Calculator | Extraction of PDF password hashes |
| Password Cracker | Networkwalks password recovery |
| `password.txt` | Custom-password wordlist |


## Reconnaissance

whois.domaintools.com is a specialized lookup service used to search DNS records, the registration databases that store ownership, technical configuration, and contact details for website domain names.


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20221530.png?raw=true)


whatweb is a popular web scanner used in cybersecurity, penetration testing, and web reconnaissance to identify the technologies running on websites.


While Wafw00f is an open-source Python tool designed specifically to detect and fingerprint Web Application Firewalls (WAFs)


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20221822.png?raw=true)

Service enumeration was attempted with:

nmap -sS -sV medirozahospital.com

Running nmap -sS -sV gives you the best of both worlds: efficiency/stealth during the initial discovery phase (-sS), followed by precise intelligence gathering on what those services actually are (-sV)


HTTP reconnaissance:

curl -i https://medirozahospital.com/patient/login.php

This disclosed HTTP/application details including PHP, LiteSpeed, Mediroza CMS version information, and a session cookie — used for technology fingerprinting only; no standalone finding is raised solely on version disclosure.



![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20222158.png?raw=true)

## Findings

### F-01 — SQL Injection Leading to Patient Portal Authentication Bypass


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20222422.png?raw=true)


**Severity: CRITICAL**

| Attribute | Details |
|---|---|
| Affected Component | `https://medirozahospital.com/patient/login.php` |
| Category | SQL Injection / Authentication Bypass |
| CWE | CWE-89 — Improper Neutralization of Special Elements used in an SQL Command |
| Primary Impact | Unauthorized access to confidential patient laboratory reports |


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20222349.png?raw=true)


**Description:** The patient portal login was vulnerable to SQL injection. Malformed input first produced a MySQL syntax error, and the following username payload achieved a full authentication bypass:


admin'--

![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20222824.png?raw=true)



This bypassed the authentication logic entirely, granting access to the patient portal and exposing three password-protected pathology reports.



![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20222915.png?raw=true)



**Attack Chain:**
```
SQL Injection -> Authentication Bypass -> Patient Portal -> 3 Encrypted Reports
   -> PDF Hash Extraction -> Dictionary Attack -> Password Recovery -> Report Access
```

**Recommendations:**
- Use parameterized queries / prepared statements for all database queries
- Never construct SQL by concatenating user-controlled input
- Disable detailed database error messages in production; return generic auth errors
- Implement strong authentication and secure password storage
- Apply authorization checks after authentication
- Review other authentication endpoints for the same weakness
- Perform regression testing after remediation

### F-02 — Weak Password Protection on Confidential Patient Reports
**Severity: HIGH**

| Attribute | Details |
|---|---|
| Affected Components | Three password-protected patient laboratory reports |
| Category | Weak Password / Insufficient Protection of Sensitive Files |
| CWE | CWE-521 — Weak Password Requirements |
| Primary Impact | Recovery of passwords protecting sensitive patient reports |

**Description:** All three encrypted PDF reports obtained via F-01 had their password protection defeated using Networkwalks hash calculator, password cracker recovery. Hashes were extracted with hash calculator and cracked with password cracker using the custom wordlist `password.txt`. Different passwords were recovered for each report; all three opened successfully.

**Technical Procedure:**

# Extract hash from locked patient.pdf

Networkwalks hash calculator 


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20223222.png?raw=true)


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20223642.png?raw=true)




# Crack with password cracker using (password.txt) custom wordlist

Networkwalks password cracker 


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20223422.png?raw=true)

![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20223728.png?raw=true)




THE SAME PROCESS WAS REPEATED INDEPENDENTLY FOR REPORTS 2 & 3.


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20223528.png?raw=true)

![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20225335.png?raw=true)



**Impact:** The reports contained patient name and identifier, date of birth and gender, lab reference/specimen data, referring physician information, clinical test results and reference ranges, and abnormality flags — a loss of confidentiality.

![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20223548.png?raw=true)


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20223556.png?raw=true)


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20225419.png?raw=true)



**Recommendations:**
- Use strong, randomly generated passwords for sensitive PDFs
- Avoid deriving document passwords from predictable patient information
- Prefer application-level authorization and secure document delivery over distributed passwords
- Use stronger document encryption mechanisms
- Review access controls for all sensitive documents

### F-03 — Unauthenticated Exposure of Internal Database Backup
**Severity: CRITICAL**

| Attribute | Details |
|---|---|
| Affected Resource | `https://medirozahospital.com/old/` |
| Exposed File | `/old/mediroza_db_backup_2019.sql` |
| Category | Sensitive Information Exposure / Exposure of Backup File |
| CWE | CWE-530 — Exposure of Backup File |
| Primary Impact | Unauthorized disclosure of internal staff and shareholder records |


![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-02%20024756.png?raw=true)

**Discovery Chain:**

robots.txt -> /old/ -> directory listing -> mediroza_db_backup_2019.sql -> unauthenticated access


**Description:** `robots.txt` disclosed the `/old/` path (alongside `/patient/` and `/staff/`). Accessing `/old/` revealed a public directory listing containing a SQL database backup, viewable directly through the browser without any authentication.

> **Note:** `robots.txt` is not an access-control mechanism — the impact arises because the disclosed `/old/` resource was itself publicly accessible.

**Data exposed in the backup:**

*Staff table:*
- Full name, job title, department
- Email address and telephone number
- National identification number
- Monthly salary
- Date joined

*Shareholders table:*
- Shareholder name
- Share percentage
- Shares held
- Share class
  
![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20230718.png?raw=true)

**Impact:** Unauthorized disclosure of employee PII, contact and national ID information, employment and salary data, and shareholder/ownership information — directly accessible over HTTP(S) with no authentication, and useful for follow-on targeted attacks.

![Image Alt](https://github.com/Cyberhunter1289/NETWORKWALKS-B083-WK4-PM1-CYBERSECURITY-BLACKBOX--PENETRATION-TESTING-ON-MEDIROZAHOSPITAL.COM/blob/main/Screenshot%202026-10-01%20230748.png?raw=true)

**Recommendations:**
- Remove database backups from publicly accessible web directories
- Store backups outside the web server document root
- Implement strict access controls for backup files
- Disable directory listing on the web server
- Remove obsolete/legacy directories such as `/old/`
- Audit the web root for exposed `.sql`, `.bak`, `.zip`, `.tar` and similar files
- Establish secure backup storage procedures
- Prevent backup files from being served over HTTP/HTTPS
- Perform periodic external checks for exposed backup files

## Overall Risk Summary

| ID | Finding | Severity | Confidentiality Impact | Priority |
|---|---|---|---|---|
| F-01 | SQL Injection -> Patient Portal Authentication Bypass | Critical | Severe | Immediate |
| F-02 | Weak Password Protection on Patient Reports | High | High | High |
| F-03 | Unauthenticated Database Backup Exposure | Critical | Severe | Immediate |

**Prioritization:** Remediate F-01 and F-03 immediately. F-02 should follow as a high-priority remediation since it directly weakens protection of already-sensitive patient reports.

## Recommendations

### Priority 1 — Immediate
- Fix the SQL injection with parameterized queries across all authentication endpoints
- Remove `/old/mediroza_db_backup_2019.sql` from the public web root
- Disable directory listing site-wide

### Priority 2 — High
- Replace weak/static PDF passwords with strong secrets or application-level authorization for report delivery
- Disable verbose database error disclosure in production

### Priority 3 — Ongoing
- Store backups outside the web root, encrypted at rest, with restricted access
- Periodically audit the web root for legacy/forgotten resources and exposed backup files
- Conduct a validation re-test after remediation

---

## Evidence Handling & Redaction

This engagement involved real (simulated) patient, staff, and shareholder data categories. In line with the original report's guidance:

- Evidence containing patient, employee, or shareholder information should be redacted before any distributed version of a report is shared.
- Recovered PDF passwords are intentionally omitted from this summary.
- Original unredacted evidence should be retained only in an authorized, access-controlled evidence repository.

---

## Conclusion

The assessment identified significant weaknesses across input validation, SQL query construction, authentication controls, sensitive-data access controls, document password management, web-server file exposure, backup management, and legacy resource handling. The Critical SQL injection and database backup exposure findings should be remediated first, followed by the High weak-password-protection finding. A validation assessment is recommended after remediation to confirm the issues have been resolved.

---

## Disclaimer

This report was prepared solely for Mediroza General Hospital in connection with an authorized penetration testing and vulnerability assessment. Testing was restricted to the target domain; social engineering and denial-of-service testing were explicitly excluded. This document contains security-sensitive assessment information and should be handled as confidential.


**Tags:** `penetration-testing` `web-security` `sql-injection` `authentication-bypass` `information-disclosure` `password-cracking` `hash calculator` `pdf-security` `vulnerability-assessment`

*Prepared by OPEYEMI AROWOSAFE · Report Date: 01 October 2026*

