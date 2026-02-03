---
title: 'LockBit 3.0 Ransomware via Phishing Email'
date: '2024-08-15T00:00:00-05:00'
draft: false

# Incident Metadata
severity: "critical"
date_detected: '2024-08-15T14:23:17-05:00'
date_resolved: '2024-08-22T18:00:00-05:00'
incident_type: "Ransomware"

# MITRE ATT&CK Framework
technique_used: ["T1566.001", "T1204.002", "T1059.001", "T1047", "T1486", "T1490", "T1070.001", "T1021.002"]
tactics: ["Initial Access", "Execution", "Persistence", "Defense Evasion", "Lateral Movement", "Impact"]

# Investigation Details
affected_systems: "1 initial workstation (ACME-WKS-0247), 3 file servers (FS01, FS02, FS03), 47 mapped network drives"
initial_vector: "Spearphishing attachment via personal webmail accessed on corporate workstation"
root_cause: "Lack of email filtering on personal webmail, insufficient endpoint protection, overly permissive network share access"

# Artifacts & Telemetry
key_artifacts: ["Windows Event Logs (Security, System, Application)", "Prefetch Files", "$MFT Timeline", "Email Attachment (malicious ZIP)", "Memory Dump", "Network Traffic Captures", "Volume Shadow Copies", "Registry Hives"]
iocs: ["Invoice_Aug2024.zip (SHA256: 7f3e9a2b1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f)", "lockbit.exe (SHA256: 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b)", "C2: 185.220.101.47:8443", "TOR Hidden Service: http://lockbit...onion"]

# Case Study Metadata
sanitized: true
lessons_learned: "Personal webmail access on corporate assets is a critical blind spot. Network segmentation and least-privilege access would have limited impact. EDR detection failed due to obfuscation techniques."
tags: ["ransomware", "lockbit", "phishing", "lateral-movement", "windows-forensics"]
---

## Executive Summary

On August 15, 2024, Acme Co. experienced a LockBit 3.0 ransomware attack that encrypted approximately 2.3TB of data across three file servers and 47 mapped network drives. The initial infection vector was a malicious ZIP attachment opened by an employee accessing personal Gmail on their corporate workstation (ACME-WKS-0247).

The adversary achieved initial execution at **13:47 UTC**, established persistence, disabled Windows Defender and Volume Shadow Copy Service, and began encryption at **14:18 UTC**. The attack was detected 5 minutes into the encryption phase when multiple users reported inaccessible files.

**Impact:**
- 2.3TB of data encrypted (approximately 340,000 files)
- 3 file servers taken offline
- 127 employees unable to access critical business data
- Estimated business disruption: $180,000 (7 days partial operations)

**Outcome:**
- No ransom paid
- Data restored from backups (RPO: 12 hours)
- Full operations restored within 7 days
- Zero data exfiltration confirmed

---

## Initial Detection

**Timeline of Detection:**

**14:23:17 UTC** - Multiple help desk tickets submitted reporting "files won't open" and "strange file extensions (.lockbit)"

**14:24:03 UTC** - SOC analyst noticed spike in SMB traffic from ACME-WKS-0247 to file servers

**14:25:41 UTC** - Automated alert: Abnormal file modification rate detected on FS01 (12,000+ files/minute)

**14:26:15 UTC** - Incident declared; IR team activated

**Telemetry Sources:**
1. **SIEM Alert**: Abnormal file modification velocity (Splunk correlation rule)
2. **EDR Alert**: Suspicious PowerShell execution with obfuscated commands (CrowdFalcon)
3. **User Reports**: Multiple simultaneous tickets via ServiceNow
4. **Network Monitoring**: Unusual SMB traffic patterns (Zeek logs)

**Initial Triage:**
- Ransom note (`README_TO_DECRYPT.txt`) found on user desktop and all encrypted directories
- File extensions changed to `.lockbit`
- Wallpaper changed to LockBit ransom message
- Network shares inaccessible

---

## Investigation Timeline

### Phase 1: Containment (14:26 - 15:00 UTC)

**14:26** - Isolated ACME-WKS-0247 from network (switch port disabled)  
**14:28** - Disabled user account `jsmith` in Active Directory  
**14:30** - Took FS01, FS02, FS03 offline to prevent further encryption  
**14:35** - Initiated memory dump of ACME-WKS-0247 before shutdown  
**14:42** - Verified backups integrity (last successful backup: 02:00 UTC same day)  
**14:50** - Confirmed no additional compromised systems via EDR sweep  
**14:58** - Network traffic analysis: no ongoing C2 communication detected  

### Phase 2: Forensic Collection (15:00 - 18:00 UTC)

**15:15** - Disk image acquired from ACME-WKS-0247 (EnCase FTK Imager)  
**15:45** - Collected Windows Event Logs from affected file servers  
**16:20** - Extracted email from user's Outlook cache (Gmail IMAP sync enabled)  
**16:55** - Captured network traffic logs from firewall and IDS (last 24 hours)  
**17:30** - Collected registry hives and prefetch files from workstation  
**17:45** - Preserved Volume Shadow Copies from file servers (2 copies available)  

### Phase 3: Analysis (18:00 - 23:00 UTC, Day 1)

**18:30** - Identified malicious ZIP attachment in Gmail cache  
**19:15** - Extracted and analyzed `Invoice_Aug2024.zip` (contained obfuscated JavaScript)  
**20:00** - Timeline analysis using Plaso (supertimeline generated)  
**21:10** - Identified initial execution timestamp: 13:47:22 UTC  
**22:00** - Reconstructed attack chain from prefetch, registry, and event logs  
**22:45** - Confirmed LockBit 3.0 variant via binary analysis  

### Phase 4: Recovery (Day 2-7)

**Day 2** - Rebuilt ACME-WKS-0247 from gold image  
**Day 3-5** - Restored file servers from backups (2.3TB data)  
**Day 6** - Validation testing with business units  
**Day 7** - Full production restoration; incident closed  

---

## Technical Analysis

### Key Artifacts Examined

#### 1. Malicious Email Attachment

**File:** `Invoice_Aug2024.zip`  
**SHA256:** `7f3e9a2b1c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f`  
**Contained:** `Invoice_Aug2024.js` (obfuscated JavaScript)  

**Analysis:**
- JavaScript used WScript.Shell to download second-stage payload
- Download URL: `hxxp://185.220.101.47/update.exe`
- Heavy obfuscation using string concatenation and character code manipulation

#### 2. Windows Event Logs

**Event ID 4688 (Process Creation):**
```
13:47:22 - wscript.exe (PID 3428) executed Invoice_Aug2024.js
13:47:35 - powershell.exe (PID 4192) spawned by wscript.exe
13:47:41 - lockbit.exe (PID 4856) created in C:\Users\jsmith\AppData\Local\Temp\
```

**Event ID 4672 (Special Privileges Assigned):**
```
13:48:15 - SeDebugPrivilege assigned to lockbit.exe
```

**Event ID 7045 (Service Installation):**
```
13:49:03 - Service "WindowsUpdateCheck" installed (binary: lockbit.exe)
```

#### 3. Prefetch Analysis

**Files Examined:**
- `WSCRIPT.EXE-A3F1D2E4.pf` - Execution count: 1, Last run: 13:47:22 UTC
- `POWERSHELL.EXE-B4C2A1F3.pf` - Execution count: 3, Last run: 13:47:35 UTC
- `LOCKBIT.EXE-C5D3B2E1.pf` - Execution count: 1, Last run: 13:47:41 UTC

**Key Finding:** Prefetch confirmed single execution of malicious JavaScript, indicating no prior compromise.

#### 4. $MFT Timeline Analysis

**Critical Timestamps:**
```
13:47:22 - Invoice_Aug2024.js created in Downloads folder
13:47:41 - lockbit.exe created in %TEMP%
13:48:30 - Registry modifications (Run key persistence)
13:52:15 - vssadmin.exe executed (VSS deletion)
14:18:03 - Mass file encryption begins (first .lockbit extension observed)
```

#### 5. Memory Forensics (Volatility 3)

**Process Tree:**
```
explorer.exe (PID 2184)
  └─ wscript.exe (PID 3428)
      └─ powershell.exe (PID 4192)
          └─ lockbit.exe (PID 4856)
              └─ vssadmin.exe (PID 5012)
```

**Network Connections:**
```
lockbit.exe (PID 4856) -> 185.220.101.47:8443 (ESTABLISHED)
```

**Injected Code:** Process hollowing detected in `svchost.exe` (PID 1832)

#### 6. Registry Analysis

**Persistence Mechanism:**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
Value: "WindowsSecurityUpdate"
Data: "C:\Users\jsmith\AppData\Local\Temp\lockbit.exe"
```

**Defender Exclusions Added:**
```
HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths
Value: "C:\Users\jsmith\AppData\Local\Temp\"
```

#### 7. Volume Shadow Copy Analysis

**Finding:** All VSS copies deleted via `vssadmin.exe Delete Shadows /All /Quiet`

**Event ID 8222 (VSS Deletion):**
```
13:52:15 - Shadow copies deleted by vssadmin.exe (PID 5012)
```

---

### Attack Chain Reconstruction

#### Stage 1: Initial Access (13:47:22 UTC)
1. User `jsmith` accessed personal Gmail via Chrome browser
2. Opened email with subject "Invoice Payment Overdue - Urgent"
3. Downloaded attachment `Invoice_Aug2024.zip`
4. Extracted and double-clicked `Invoice_Aug2024.js`

#### Stage 2: Execution (13:47:35 UTC)
1. JavaScript executed via `wscript.exe`
2. PowerShell spawned with `-ExecutionPolicy Bypass -WindowStyle Hidden`
3. Downloaded `lockbit.exe` from `185.220.101.47`
4. Saved to `C:\Users\jsmith\AppData\Local\Temp\lockbit.exe`

#### Stage 3: Persistence (13:48:30 UTC)
1. Registry Run key created for auto-start
2. Service "WindowsUpdateCheck" installed
3. Scheduled task created (backup persistence)

#### Stage 4: Defense Evasion (13:49:15 UTC)
1. Windows Defender disabled via registry modification
2. Exclusion paths added to Defender configuration
3. Event log clearing attempted (failed due to permissions)

#### Stage 5: Privilege Escalation (13:50:00 UTC)
1. UAC bypass via `fodhelper.exe` registry hijack
2. Elevated process spawned with SYSTEM privileges

#### Stage 6: Discovery (13:50:30 UTC)
1. Network share enumeration via `net view` and `net use`
2. Active Directory query for file server locations
3. SMB connection testing to FS01, FS02, FS03

#### Stage 7: Lateral Movement (13:55:00 UTC)
1. Mapped network drives accessed using user's cached credentials
2. Ransomware binary copied to `\\FS01\C$\Windows\Temp\`
3. Remote execution via WMI (`wmic` command)

#### Stage 8: Impact - VSS Deletion (13:52:15 UTC)
1. `vssadmin.exe Delete Shadows /All /Quiet` executed
2. All Volume Shadow Copies destroyed
3. Windows Backup catalog deleted

#### Stage 9: Impact - Encryption (14:18:03 UTC)
1. Encryption started on local drives (C:, D:)
2. Network shares encrypted via SMB (FS01, FS02, FS03)
3. File extensions changed to `.lockbit`
4. Ransom note dropped in every directory
5. Wallpaper changed to ransom message

---

## Root Cause Analysis

### Primary Root Cause
**Unrestricted personal webmail access on corporate workstations** combined with **insufficient email security controls** allowed the initial infection vector.

### Contributing Factors

1. **Policy Violation:** Corporate policy prohibited personal email access, but technical controls were not enforced
2. **Endpoint Protection Gap:** Windows Defender was the sole AV solution; no EDR with behavioral detection
3. **Overly Permissive Network Shares:** User `jsmith` had write access to all file servers despite role not requiring it
4. **Lack of Network Segmentation:** Workstations had direct SMB access to file servers
5. **Insufficient User Training:** User did not recognize phishing indicators (external sender, urgency, unexpected attachment)
6. **No Email Sandboxing:** Personal webmail attachments bypassed corporate email gateway protections

### Security Control Failures

| Control | Expected Behavior | Actual Behavior | Impact |
|---------|------------------|-----------------|--------|
| Web Proxy | Block personal webmail | Not configured | Allowed Gmail access |
| Email Gateway | Scan attachments | Bypassed (webmail) | No scanning occurred |
| Endpoint AV | Detect malicious JS | Failed (obfuscation) | Execution allowed |
| Application Whitelisting | Block unauthorized executables | Not deployed | lockbit.exe executed |
| Least Privilege | Restrict file server access | Not enforced | Lateral movement enabled |
| Network Segmentation | Isolate workstations from servers | Not implemented | Direct SMB access allowed |

---

## Containment & Eradication

### Immediate Actions (First Hour)

1. **Network Isolation**
   - Disabled switch port for ACME-WKS-0247
   - Blocked IP `185.220.101.47` at firewall
   - Disabled user account `jsmith`

2. **System Shutdown**
   - Gracefully shut down file servers to preserve memory
   - Prevented additional encryption

3. **Backup Verification**
   - Confirmed backup integrity (last backup: 02:00 UTC)
   - Verified backups not encrypted (offline storage)
   - Calculated RPO: 12 hours of data loss

### Forensic Preservation

1. **Memory Dumps:** Captured RAM from ACME-WKS-0247 before shutdown
2. **Disk Images:** Full forensic images of workstation and file servers
3. **Log Collection:** Centralized all event logs, network logs, and EDR telemetry
4. **Email Preservation:** Exported user's mailbox for evidence

### Eradication Steps

1. **Workstation Rebuild**
   - Wiped ACME-WKS-0247 completely
   - Rebuilt from trusted gold image
   - Applied all security patches

2. **File Server Restoration**
   - Restored FS01, FS02, FS03 from backups
   - Verified no malware in restored data
   - Applied latest Windows updates

3. **Credential Reset**
   - Forced password reset for `jsmith`
   - Rotated service account passwords
   - Invalidated all Kerberos tickets

4. **Network Cleanup**
   - Removed malicious scheduled tasks across domain
   - Scanned all systems for IOCs (none found)
   - Verified no additional persistence mechanisms

---

## Remediation & Hardening

### Immediate Remediations (Week 1)

1. **Web Proxy Configuration**
   - Blocked personal webmail (Gmail, Yahoo, Outlook.com)
   - Implemented SSL inspection for HTTPS traffic
   - Created exception process for business justification

2. **Endpoint Protection Upgrade**
   - Deployed CrowdStrike Falcon EDR to all workstations
   - Enabled behavioral detection and machine learning
   - Configured automatic containment for ransomware indicators

3. **Network Share Permissions**
   - Audited all file server permissions
   - Implemented least-privilege access model
   - Removed `jsmith` write access to unnecessary shares

4. **Application Whitelisting**
   - Deployed AppLocker policies
   - Blocked script execution from user-writable directories
   - Whitelisted only approved applications

### Long-Term Hardening (Month 1-3)

1. **Network Segmentation**
   - Implemented VLANs separating workstations from servers
   - Deployed firewall rules requiring authentication for SMB
   - Restricted lateral movement via micro-segmentation

2. **Email Security Enhancement**
   - Deployed Proofpoint email gateway with sandbox analysis
   - Configured attachment blocking for `.js`, `.vbs`, `.hta` files
   - Implemented DMARC, SPF, and DKIM validation

3. **Backup Strategy Improvement**
   - Implemented 3-2-1 backup strategy
   - Deployed immutable backups (air-gapped)
   - Reduced RPO to 4 hours with incremental backups

4. **User Awareness Training**
   - Mandatory phishing awareness training (KnowBe4)
   - Monthly simulated phishing campaigns
   - Incident reporting incentive program

5. **Monitoring & Detection**
   - Deployed file integrity monitoring (FIM) on critical shares
   - Created SIEM correlation rules for ransomware indicators
   - Implemented honeypot files on file servers (canary tokens)

6. **Privileged Access Management**
   - Deployed CyberArk for privileged account management
   - Implemented just-in-time (JIT) admin access
   - Enforced MFA for all administrative actions

---

## Lessons Learned

### What Went Well

1. **Backup Strategy:** Offline backups prevented total data loss; 12-hour RPO was acceptable
2. **Incident Response Speed:** Detection to containment in 3 minutes prevented wider spread
3. **Team Coordination:** IR team, IT, and business units worked seamlessly
4. **Forensic Preservation:** Proper evidence collection enabled thorough root cause analysis

### What Went Wrong

1. **Policy Enforcement Gap:** Personal webmail policy existed but wasn't technically enforced
2. **Endpoint Protection Inadequacy:** Signature-based AV failed against obfuscated malware
3. **Excessive Permissions:** User had unnecessary write access to critical file servers
4. **Detection Delay:** 31 minutes elapsed between initial execution and detection

### What Would I Do Differently?

1. **Proactive Threat Hunting:** Regular hunts for anomalous PowerShell and script execution
2. **Faster Isolation:** Automated EDR containment would have prevented lateral movement
3. **Email Sandboxing:** Even personal webmail attachments should be inspected via proxy
4. **Network Segmentation:** Should have been in place before incident; prevented lateral movement

### Key Takeaways

> **Personal webmail is a critical blind spot.** If you can't block it, you must inspect it.

> **Least privilege is not optional.** Every user with excessive permissions is a potential pivot point.

> **Backups are your last line of defense.** Air-gapped, immutable backups saved this organization $500K+ in ransom and recovery costs.

> **EDR > AV.** Behavioral detection would have caught the PowerShell execution and process injection immediately.

---

## MITRE ATT&CK Mapping

### Detailed Technique Analysis

| Tactic | Technique ID | Technique Name | Evidence |
|--------|--------------|----------------|----------|
| **Initial Access** | T1566.001 | Phishing: Spearphishing Attachment | Malicious ZIP attachment via Gmail |
| **Execution** | T1204.002 | User Execution: Malicious File | User double-clicked `Invoice_Aug2024.js` |
| **Execution** | T1059.001 | Command and Scripting Interpreter: PowerShell | PowerShell used to download payload |
| **Persistence** | T1547.001 | Boot or Logon Autostart Execution: Registry Run Keys | `HKCU\...\Run` key created |
| **Persistence** | T1543.003 | Create or Modify System Process: Windows Service | "WindowsUpdateCheck" service installed |
| **Defense Evasion** | T1562.001 | Impair Defenses: Disable or Modify Tools | Windows Defender disabled |
| **Defense Evasion** | T1070.001 | Indicator Removal: Clear Windows Event Logs | Attempted (failed due to permissions) |
| **Privilege Escalation** | T1548.002 | Abuse Elevation Control Mechanism: Bypass UAC | `fodhelper.exe` registry hijack |
| **Discovery** | T1083 | File and Directory Discovery | Network share enumeration |
| **Discovery** | T1135 | Network Share Discovery | `net view` and `net use` commands |
| **Lateral Movement** | T1021.002 | Remote Services: SMB/Windows Admin Shares | Accessed `\\FS01\C$\` |
| **Lateral Movement** | T1047 | Windows Management Instrumentation | WMI used for remote execution |
| **Impact** | T1486 | Data Encrypted for Impact | LockBit ransomware encryption |
| **Impact** | T1490 | Inhibit System Recovery | VSS deletion via `vssadmin.exe` |

### Attack Flow Diagram

```
[T1566.001] Phishing Email
      ↓
[T1204.002] User Opens Attachment
      ↓
[T1059.001] PowerShell Downloads Payload
      ↓
[T1547.001] Registry Persistence
      ↓
[T1562.001] Disable Defender
      ↓
[T1548.002] UAC Bypass
      ↓
[T1135] Network Share Discovery
      ↓
[T1021.002] Lateral Movement via SMB
      ↓
[T1490] Delete Volume Shadow Copies
      ↓
[T1486] Ransomware Encryption
```

### Detection Opportunities by Technique

| Technique | Detection Method | Implemented? |
|-----------|------------------|--------------|
| T1566.001 | Email gateway sandbox analysis | ❌ (bypassed via webmail) |
| T1059.001 | PowerShell script block logging + SIEM correlation | ⚠️ (logged but not alerted) |
| T1562.001 | Registry monitoring for Defender modifications | ❌ |
| T1490 | Event ID 8222 (VSS deletion) alert | ✅ (post-incident) |
| T1486 | File modification velocity monitoring | ✅ (triggered alert) |

---

**Investigation Lead:** Patrick Jones  
**Report Date:** August 23, 2024  
**Classification:** Sanitized Case Study
