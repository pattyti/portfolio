---
title: "/cases"
date: 2026-02-02T00:00:00-05:00
draft: false
---

# Incident Case Studies

This section contains sanitized, real-world incident investigations from my career in DFIR and SOC operations. Each case study is structured to provide:

* **Technical depth:** Artifact-level analysis, not just high-level summaries
* **Methodology:** The investigative process, including dead ends and pivots
* **MITRE ATT&CK mapping:** Precise technique identification for threat modeling
* **Lessons learned:** What worked, what didn't, and how to improve

---

## Investigation Frameworks by Incident Type

Before diving into specific cases, understanding the investigative approach for different incident categories is critical. Each incident type has unique artifacts, telemetry sources, and potential pitfalls.

### Ransomware Investigations
**Priority Artifacts:**
* Volume Shadow Copies (often deleted by adversary)
* Windows Event Logs (Security, System, Application)
* Prefetch files (execution timeline)
* $MFT and $UsnJrnl (file system timeline)
* Memory dumps (encryption keys, process injection)

**Critical Questions:**
1. What was the initial access vector?
2. How long was the adversary present before encryption?
3. Were credentials compromised?
4. Was data exfiltrated before encryption?

**Common Pitfalls:**
* Focusing only on the ransomware binary, missing the initial compromise
* Assuming encryption timestamp = breach timestamp
* Overlooking lateral movement artifacts

---

### Data Exfiltration
**Priority Artifacts:**
* Network traffic captures (PCAP analysis)
* Proxy logs and DNS queries
* Cloud storage access logs
* USB device connection history (USBSTOR registry)
* Email server logs

**Critical Questions:**
1. What data was accessed?
2. What was the exfiltration method? (Cloud storage, email, removable media, C2 channel)
3. Was the exfiltration automated or manual?
4. How was access to sensitive data obtained?

**Common Pitfalls:**
* Relying solely on DLP alerts (often incomplete)
* Missing encrypted exfiltration channels
* Failing to identify the full scope of compromised data

---

### Insider Threats
**Priority Artifacts:**
* User activity logs (VPN, authentication, file access)
* Email and chat communications (legal approval required)
* Database query logs
* Print spooler logs
* Physical access logs

**Critical Questions:**
1. Was this malicious or negligent?
2. What was the user's access level and normal behavior baseline?
3. Were credentials shared or compromised?
4. What was the timeline of suspicious activity?

**Common Pitfalls:**
* Confirmation bias (assuming guilt)
* Insufficient baseline of normal user behavior
* Privacy and legal considerations not properly addressed

---

### Business Email Compromise (BEC)
**Priority Artifacts:**
* Email headers (full RFC 822 format)
* Mail server logs (Exchange, O365)
* Mailbox rules and forwarding configurations
* Authentication logs (MFA bypass attempts)
* Financial transaction records

**Critical Questions:**
1. Was the account compromised or spoofed?
2. Were mailbox rules created for persistence?
3. What was the social engineering technique?
4. How many users were targeted?

**Common Pitfalls:**
* Focusing only on the final fraudulent email
* Missing mailbox rule persistence mechanisms
* Inadequate analysis of email routing and authentication

---

### Web Application Compromise
**Priority Artifacts:**
* Web server access logs (Apache, Nginx, IIS)
* Application logs
* Database query logs
* WAF logs
* File integrity monitoring alerts

**Critical Questions:**
1. What vulnerability was exploited?
2. Was a web shell deployed?
3. What level of access did the adversary achieve?
4. Was the database compromised?

**Common Pitfalls:**
* Log rotation destroying critical evidence
* Focusing on the exploit, missing post-exploitation activity
* Insufficient understanding of application architecture

---

## Case Studies

Below are detailed sample investigations from my career in DFIR and SOC operations. These are not real incidents, but rather simulations of real incidents.

<!-- Individual case studies will be added here -->
