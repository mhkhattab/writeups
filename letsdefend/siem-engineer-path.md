# LetsDefend — SIEM Engineer Career Path

## Overview

**Platform:** LetsDefend  
**Role focus:** SIEM Engineering / Detection Infrastructure  
**Core areas:** SIEM architecture, capacity planning, collection, parsing, normalization, alerting, search/reporting, Splunk, Wazuh, tuning, SOAR, TheHive

This writeup focuses on the engineering work required to turn raw telemetry into reliable detections. It intentionally excludes quiz answers and platform-specific assessment keys.

## Current Path Structure

The official path includes:

1. SIEM 101
2. Network Security
3. SIEM Basics, Installation and Configuration
4. SIEM Log Collection and Parsing
5. SIEM Alert Generation
6. SIEM Log Search, Analysis and Reporting
7. Splunk
8. Wazuh
9. SIEM Alert Tuning
10. SOAR Fundamentals
11. Open Source SOAR: TheHive
12. Advanced SOAR

## 1. SIEM Engineering Lifecycle

```text
Requirements
    ↓
Data-source inventory
    ↓
Collection architecture
    ↓
Transport
    ↓
Parsing
    ↓
Normalization
    ↓
Storage / retention
    ↓
Detection logic
    ↓
Testing
    ↓
Deployment
    ↓
Tuning
    ↓
Health monitoring
```

A detection platform is only as trustworthy as its data pipeline.

## 2. Architecture and Capacity Planning

Before installing a SIEM, identify:

- event sources
- average EPS
- burst EPS
- average event size
- daily ingest volume
- retention requirements
- replication factor
- search concurrency
- regulatory requirements
- disaster-recovery objectives

A rough ingestion estimate:

```text
daily_bytes =
  events_per_second
  × average_event_size
  × 86,400
```

Then account for:

- indexing overhead
- replicas
- hot/warm/cold tiers
- growth
- burst tolerance

## 3. Data Collection Methods

### Syslog

Common for:

- firewalls
- routers
- switches
- Linux services
- security appliances

### Agents

Useful for richer host telemetry:

- Windows Event Logs
- Sysmon
- file integrity
- endpoint events
- local application logs

### APIs

Common for:

- SaaS
- cloud control planes
- EDR
- identity providers
- Microsoft 365

### Files / databases

Some legacy applications require:

- file tailing
- JDBC/ODBC
- custom collectors

Collection health must be monitored. "No alerts" can mean "no telemetry."

## 4. Parsing

Raw example:

```text
Oct 15 10:04:03 fw01 action=deny src=10.0.0.15 dst=203.0.113.20 dpt=443
```

Desired normalized fields:

```text
timestamp
device_name
action
source_ip
destination_ip
destination_port
```

Regex example:

```regex
src=(?<source_ip>\d{1,3}(?:\.\d{1,3}){3})
```

Production parsers should be tested against:

- missing fields
- reordered fields
- malformed events
- multiline data
- timezone differences
- vendor-version changes

## 5. Normalization

Without normalization, detections become vendor-specific.

For example:

```text
src
src_ip
sourceAddress
client_ip
```

should map into a consistent field model when they represent the same semantic value.

Useful normalization domains:

- source/destination IP
- user
- hostname
- process
- action
- event category
- authentication result

## 6. Time Integrity

Correlation depends on accurate time.

Control:

- NTP
- timezones
- DST handling
- event timestamp vs ingestion timestamp
- clock skew
- parser timestamp formats

A five-minute clock drift can break multi-source incident timelines.

## 7. Detection Engineering

A detection rule should define:

- hypothesis
- required telemetry
- field assumptions
- selection
- aggregation
- threshold
- time window
- exclusions
- severity
- ATT&CK mapping
- analyst guidance

Example brute-force logic:

```text
authentication failures >= threshold
grouped by user and source
within 5 minutes
```

Then enrich the detection with context:

- privileged account?
- successful login afterward?
- known scanner?
- internal test source?
- impossible travel?

## 8. Search and Reporting

Search performance is an engineering concern.

Review:

- indexed fields
- wildcard use
- time range
- cardinality
- expensive joins
- dashboard refresh intervals
- report schedules

Reports should depend on normalized fields rather than fragile raw-string matches.

## 9. Splunk

Core architecture concepts:

- Universal Forwarder
- Heavy Forwarder
- Indexer
- Search Head
- Index
- Sourcetype
- field extraction
- knowledge objects

Example SPL:

```spl
index=authentication action=failure
| stats count dc(user) as distinct_users by src_ip
| where count > 20
| sort - count
```

Time-series view:

```spl
index=authentication
| timechart span=5m count by action
```

A good sourcetype and parser design makes detection content reusable.

## 10. Wazuh

A simplified Wazuh event flow:

```text
Agent
  ↓
Manager
  ↓
Decoder
  ↓
Normalized fields
  ↓
Rule
  ↓
Alert
```

Engineering tasks include:

- agent enrollment
- connectivity validation
- decoder development
- rule creation
- file-integrity configuration
- vulnerability/security configuration monitoring
- alert integration

Decoder testing must happen before rule tuning. A bad parser cannot produce a reliable detection.

## 11. Alert Tuning

Tuning is not "turning alerts off."

Classify noisy events as:

- false positive
- benign true positive
- duplicate
- expected administrative behavior
- parser defect
- threshold problem

Safe process:

```text
collect examples
→ identify benign pattern
→ define narrow exception
→ regression-test malicious behavior
→ deploy
→ monitor
```

Avoid broad suppressions such as "ignore PowerShell."

## 12. Detection-as-Code

Treat detection content as engineering artifacts.

Store:

- rule
- version
- owner
- data-source requirements
- ATT&CK mapping
- test fixtures
- positive tests
- negative tests
- changelog

Example validation cases:

```text
Known malicious event → MUST alert
Known benign event    → MUST NOT alert
Boundary threshold    → deterministic expected result
Malformed event       → parser failure visible
```

## 13. SOAR Fundamentals

Good automation targets repetitive and low-risk steps:

- IOC enrichment
- WHOIS/reputation queries
- case creation
- context gathering
- notifications

High-impact actions require stronger controls:

- disable user
- isolate endpoint
- block domain
- delete email

Design principles:

- idempotency
- retries
- timeout handling
- audit logging
- secrets management
- human approval
- rollback

## 14. TheHive

TheHive can structure:

- alerts
- cases
- observables
- tasks
- ownership
- timelines
- investigation notes

A typical integration:

```text
SIEM alert
   ↓
SOAR enrichment
   ↓
TheHive case
   ↓
Analyst validation
   ↓
Approved response
```

## 15. SIEM Health Monitoring

Monitor the platform itself.

Key health indicators:

- ingest lag
- dropped events
- parser failure rate
- agent disconnects
- index/storage utilization
- query latency
- rule execution errors
- clock skew
- queue backlog

A SIEM that cannot detect its own telemetry failure creates dangerous blind spots.

## 16. Engineering Validation Checklist

### Collection
- Are expected sources reporting?
- Is delivery reliable?
- Is loss visible?

### Parsing
- Are required fields extracted?
- Are malformed events quarantined?
- Are vendor updates tested?

### Detection
- Do positive fixtures alert?
- Do negative fixtures stay quiet?
- Are thresholds deterministic?

### Access
- Is RBAC least privilege?
- Are administrative actions audited?
- Are service credentials protected?

### Retention
- Are compliance requirements met?
- Are storage tiers sized correctly?
- Can historical data still be searched?

## Example Detection Record

```markdown
# Detection — Repeated Authentication Failure

## Hypothesis
An attacker is attempting password guessing.

## Data Sources
Identity provider / Windows authentication logs.

## Required Fields
- user
- source_ip
- action
- timestamp

## Logic
Count failures by user/source over five minutes.

## Exclusions
Only narrowly approved scanner/test sources.

## Validation
- Positive fixture:
- Negative fixture:
- Threshold boundary:

## ATT&CK
Brute Force

## Analyst Guidance
Check successful login after failures, MFA telemetry, source reputation,
account privilege, and endpoint activity.
```

## Skills Demonstrated

- SIEM architecture
- capacity planning
- log onboarding
- parsing and normalization
- search engineering
- detection engineering
- Splunk
- Wazuh
- alert tuning
- SOAR
- TheHive
- platform health monitoring

## References

- Official LetsDefend SIEM Engineer Career Path: https://app.letsdefend.io/path/siem-engineer-career-path
