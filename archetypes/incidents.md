---
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
date: '{{ .Date }}'
draft: true

# Incident Metadata
severity: "medium"  # Options: critical, high, medium, low
date_detected: '{{ .Date }}'
date_resolved: ""
incident_type: ""  # e.g., "Ransomware", "Data Exfiltration", "Insider Threat"

# MITRE ATT&CK Framework
technique_used: []  # e.g., ["T1566.001", "T1059.001"]
tactics: []  # e.g., ["Initial Access", "Execution", "Persistence"]

# Investigation Details
affected_systems: ""
initial_vector: ""
root_cause: ""

# Artifacts & Telemetry
key_artifacts: []  # e.g., ["Windows Event Logs", "Prefetch Files", "Memory Dump"]
iocs: []  # Indicators of Compromise

# Case Study Metadata
sanitized: true
lessons_learned: ""
tags: []
---

## Executive Summary
<!-- Brief overview of the incident: what happened, impact, and outcome -->

## Initial Detection
<!-- How was the incident first identified? What telemetry triggered the alert? -->

## Investigation Timeline
<!-- Chronological breakdown of the investigation process -->

## Technical Analysis
<!-- Deep-dive into artifacts, forensic evidence, and technical findings -->

### Key Artifacts Examined
<!-- Specific forensic artifacts that were critical to the investigation -->

### Attack Chain Reconstruction
<!-- Step-by-step reconstruction of the adversary's actions -->

## Root Cause Analysis
<!-- What vulnerability or gap allowed this incident to occur? -->

## Containment & Eradication
<!-- Actions taken to stop the threat and remove the adversary -->

## Remediation & Hardening
<!-- Long-term fixes and security improvements implemented -->

## Lessons Learned
<!-- What did this incident teach us? What would you do differently? -->

## MITRE ATT&CK Mapping
<!-- Detailed mapping of observed techniques to the ATT&CK framework -->
