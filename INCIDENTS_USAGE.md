# Incidents Archetype - Usage Guide

## Creating a New Incident Case Study

To create a new incident investigation using the custom archetype:

```bash
hugo new incidents/your-incident-name.md --kind incidents
```

This will generate a new markdown file with all the DFIR-specific front matter and structured sections.

## Front Matter Fields Explained

### Incident Metadata
- **severity**: `critical`, `high`, `medium`, or `low`
- **date_detected**: When the incident was first identified
- **date_resolved**: When the incident was fully remediated
- **incident_type**: E.g., "Ransomware", "Data Exfiltration", "BEC", "Insider Threat"

### MITRE ATT&CK Framework
- **technique_used**: Array of MITRE ATT&CK technique IDs
  - Example: `["T1566.001", "T1059.001", "T1486"]`
  - T1566.001 = Phishing: Spearphishing Attachment
  - T1059.001 = Command and Scripting Interpreter: PowerShell
  - T1486 = Data Encrypted for Impact
- **tactics**: Array of MITRE ATT&CK tactics
  - Example: `["Initial Access", "Execution", "Impact"]`

### Investigation Details
- **affected_systems**: Brief description of impacted infrastructure
- **initial_vector**: How the adversary gained initial access
- **root_cause**: The underlying vulnerability or gap that allowed the incident

### Artifacts & Telemetry
- **key_artifacts**: Array of critical forensic artifacts examined
  - Example: `["Windows Event Logs", "Prefetch Files", "$MFT Timeline", "Memory Dump"]`
- **iocs**: Array of Indicators of Compromise (hashes, IPs, domains, etc.)

### Case Study Metadata
- **sanitized**: Always `true` for public case studies
- **lessons_learned**: Brief summary of key takeaways
- **tags**: Array of relevant tags for categorization

## Example Front Matter (Populated)

```yaml
---
title: 'Conti Ransomware Deployment via Compromised VPN'
date: '2024-03-15T00:00:00-05:00'
draft: false

# Incident Metadata
severity: "critical"
date_detected: '2024-03-15T08:23:00-05:00'
date_resolved: '2024-03-22T16:00:00-05:00'
incident_type: "Ransomware"

# MITRE ATT&CK Framework
technique_used: ["T1078", "T1021.001", "T1059.001", "T1486", "T1490"]
tactics: ["Initial Access", "Lateral Movement", "Execution", "Impact", "Defense Evasion"]

# Investigation Details
affected_systems: "42 Windows servers, 156 workstations across 3 sites"
initial_vector: "Compromised VPN credentials (no MFA)"
root_cause: "Lack of MFA on VPN, weak password policy"

# Artifacts & Telemetry
key_artifacts: ["Windows Event Logs", "Prefetch Files", "VSS Analysis", "Network Traffic Captures", "Memory Forensics"]
iocs: ["conti.exe (SHA256: abc123...)", "C2: 192.0.2.45:8443", "ransom-note.txt"]

# Case Study Metadata
sanitized: true
lessons_learned: "MFA is non-negotiable. Network segmentation prevented total loss."
tags: ["ransomware", "conti", "vpn-compromise", "lateral-movement"]
---
```

## Content Structure

Each incident case study follows this structure:

1. **Executive Summary**: High-level overview for non-technical stakeholders
2. **Initial Detection**: How the incident was discovered
3. **Investigation Timeline**: Chronological breakdown of the investigation
4. **Technical Analysis**: Deep forensic dive
   - Key Artifacts Examined
   - Attack Chain Reconstruction
5. **Root Cause Analysis**: What allowed this to happen
6. **Containment & Eradication**: How the threat was stopped
7. **Remediation & Hardening**: Long-term fixes
8. **Lessons Learned**: Key takeaways
9. **MITRE ATT&CK Mapping**: Detailed technique mapping

## Tips for Writing Case Studies

1. **Be specific about artifacts**: Don't just say "logs" - specify Event ID 4624, Prefetch hash, etc.
2. **Show your methodology**: Include dead ends and pivots, not just the final answer
3. **Use technical terminology**: Your audience understands DFIR
4. **Sanitize thoroughly**: Remove all client-identifying information
5. **Map to ATT&CK precisely**: Use specific sub-techniques when applicable
6. **Include timelines**: Timestamps matter in forensics

## Navigation

The incidents section is accessible via the `/cases` menu item in your site navigation.
