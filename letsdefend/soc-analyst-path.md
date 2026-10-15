# LetsDefend — SOC Analyst Learning Path

## Overview

**Platform:** LetsDefend  
**Role focus:** SOC Analyst / Tier 1–2 Security Operations  
**Core areas:** alert triage, phishing analysis, web-attack detection, malware analysis, network logs, SIEM, incident management, Splunk, threat intelligence, brute-force detection

This document is a technical summary of the investigation methodology developed across the SOC Analyst learning path. It intentionally excludes quiz answers, alert-answer values, and platform-specific solution keys.

## Path Areas

The current path includes topics such as:

- SOC Fundamentals
- Cyber Kill Chain
- MITRE ATT&CK
- Phishing Email Analysis
- Detecting and Investigating Web Attacks
- SIEM Alert Investigation
- Malware Analysis Fundamentals
- Dynamic Malware Analysis
- Malicious Document Analysis
- Security Solutions
- Network Log Analysis
- SIEM 101
- Incident Management
- Splunk
- Cyber Threat Intelligence
- VirusTotal
- Brute-Force Detection
- Malware/SOC Lab Building

## 1. SOC Investigation Workflow

A repeatable workflow reduces missed evidence:

```text
Alert received
    ↓
Validate the detection
    ↓
Identify affected entities
    ↓
Collect raw telemetry
    ↓
Build timeline
    ↓
Enrich indicators
    ↓
Map adversary behavior
    ↓
Scope impact
    ↓
Verdict
    ↓
Contain / escalate / close
```

### Triage questions

For every alert:

- What rule fired?
- Which exact raw event triggered it?
- Is parsing correct?
- Which host/user/process is involved?
- What happened immediately before and after?
- Is the activity normal for this entity?
- Is there supporting telemetry?
- What is the potential impact?

## 2. Cyber Kill Chain and ATT&CK

The Cyber Kill Chain gives a high-level intrusion lifecycle:

```text
Reconnaissance
Weaponization
Delivery
Exploitation
Installation
Command and Control
Actions on Objectives
```

MITRE ATT&CK provides more granular behavior mapping.

Example:

```text
PowerShell execution
→ Command and Scripting Interpreter: PowerShell

Credential dumping
→ OS Credential Dumping
```

ATT&CK mapping should be evidence-based. Do not map a technique merely because it is common for a malware family.

## 3. Phishing Email Analysis

### Header triage

Review:

```text
From
Reply-To
Return-Path
Received
Message-ID
Authentication-Results
```

Check:

- SPF result
- DKIM result
- DMARC result
- sender/display-name mismatch
- Reply-To mismatch
- sending infrastructure
- timestamp anomalies

### URL triage

Defang before documentation:

```text
hxxps://example[.]com/path
```

Investigate:

- redirect chain
- domain age
- passive DNS
- TLS certificate
- reputation
- hosting ASN
- page purpose

### Attachments

Start with static evidence:

```bash
sha256sum suspicious_file
file suspicious_file
strings -a suspicious_file | less
```

Do not open an unknown attachment directly on an analyst workstation.

## 4. Detecting Web Attacks

### SQL Injection

Potential indicators:

```text
' OR 1=1--
UNION SELECT
information_schema
sleep(
benchmark(
```

Investigate:

- request method
- URI
- query/body
- status code
- response length
- source IP
- user agent
- adjacent requests
- whether the payload changed server behavior

### Cross-Site Scripting

Indicators can include:

```html
<script>
onerror=
onload=
javascript:
```

Remember to evaluate encoded variants.

### Command Injection

Watch for shell metacharacters and execution syntax:

```text
;
&&
||
|
$()
backticks
```

Context is essential. A pipe character in a legitimate data format is not automatically malicious.

## 5. SIEM Alert Investigation

Never investigate only the summarized alert.

Start from the raw event and validate field extraction.

Example analyst record:

```text
Alert name:
Rule:
Timestamp:
Host:
User:
Source IP:
Destination:
Process:
Raw event:
Related events:
IOC enrichment:
Verdict:
Escalation:
```

A false-positive decision should explain **why** the behavior is benign, not merely state "FP."

## 6. Malware Analysis

### Static triage

```bash
sha256sum sample
file sample
strings -a sample
```

For PE files, review:

- imports
- sections
- entry point
- suspicious strings
- embedded URLs/domains
- entropy / packer indicators
- signature information

### Dynamic analysis

Use an isolated malware-analysis environment.

Monitor:

- process tree
- command lines
- DNS
- outbound connections
- filesystem changes
- registry changes
- scheduled tasks/services
- persistence
- dropped files

Behavior becomes useful when converted into:

- IOCs
- detection logic
- containment guidance

## 7. Malicious Document Analysis

Office documents may contain:

- VBA
- embedded objects
- external templates
- DDE
- macros
- exploit chains

Workflow:

```text
hash
→ identify format
→ inspect archive/container
→ metadata
→ macro extraction
→ URL/string analysis
→ controlled sandbox
```

## 8. Network Log Analysis

Important data sources:

- DNS logs
- firewall logs
- proxy logs
- VPN logs
- IDS/IPS
- NetFlow
- PCAP

For packet captures:

```bash
tcpdump -nn -r capture.pcap
tshark -r capture.pcap
```

Useful Wireshark concepts include:

- display filters
- conversations
- follow TCP stream
- DNS query analysis
- HTTP request reconstruction
- protocol statistics

## 9. Splunk Investigation

Basic SPL patterns:

```spl
index=*
| stats count by sourcetype
```

Authentication failures:

```spl
index=auth action=failure
| stats count by user, src_ip
| sort - count
```

Time correlation:

```spl
index=*
| sort _time
```

The goal is to move from single-event searches to entity-centered timelines.

## 10. Threat Intelligence

IOC enrichment may involve:

- hashes
- domains
- IP addresses
- URLs
- certificates
- malware families

Do not let reputation replace analysis.

A useful verdict combines:

```text
reputation
+ telemetry
+ behavior
+ environment context
```

## 11. VirusTotal Usage

Useful pivot points include:

- detection ratio
- first/last seen
- related files
- passive DNS
- communicating files
- behavior
- vendor disagreement

A single vendor verdict is not enough for a high-confidence incident decision.

## 12. Brute-Force Detection

Possible patterns:

### Single-source brute force

```text
one source IP
→ many failures
→ one/few accounts
```

### Password spraying

```text
one source
→ many accounts
→ low attempts per account
```

### Credential stuffing

```text
many usernames/password attempts
→ often distributed infrastructure
→ possible success-after-failure
```

Correlate with:

- MFA events
- successful logins
- ASN/geolocation
- device identity
- impossible travel
- privileged account targeting

## 13. Incident Management

A strong incident case preserves:

- event timestamps
- analyst actions
- evidence
- containment
- communications
- severity changes
- handoffs
- closure rationale

The case should be reproducible by another analyst who was not present during the investigation.

## 14. Example Technical Case Template

```markdown
# Alert Title

## Alert Metadata
- Alert ID:
- Severity:
- Time:
- Host:
- User:
- Source:
- Destination:

## Initial Hypothesis

## Raw Evidence

## Timeline

## IOC Enrichment

## ATT&CK Mapping

## Scope and Impact

## Verdict
True Positive / False Positive / Benign True Positive

## Containment / Escalation

## Detection Improvement

## Lessons Learned
```

## Skills Demonstrated

- SIEM alert triage
- phishing analysis
- web attack investigation
- malware triage
- network telemetry analysis
- Splunk
- threat intelligence
- ATT&CK mapping
- case documentation
- incident escalation

## References

- Official LetsDefend SOC Analyst Learning Path: https://app.letsdefend.io/path/soc-analyst-learning-path
