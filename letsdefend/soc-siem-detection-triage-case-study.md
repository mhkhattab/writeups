# SOC / SIEM Detection & Triage — LetsDefend Case Study

**Environment:** Authorized LetsDefend labs  
**Tools:** Splunk, Wazuh  
**Focus:** alert validation, log analysis, authentication events, SSH brute-force investigation, SIEM triage

## What I worked on

- Used Splunk to search and analyze web and authentication-style logs, including filtering by client IP, URI/path, user, action, and time.
- Used aggregation and counting to identify repeated activity and compare request patterns.
- Reviewed Wazuh agent/syslog telemetry, alert levels, Windows/Linux event collection concepts, and custom-rule logic.
- Investigated SSH brute-force activity by reviewing repeated authentication failures, source and target context, event timing, and follow-on activity.
- Practiced validating raw events before trusting parsed alert fields and documenting analyst verdicts with supporting evidence.

## Investigation workflow

1. Validate the triggering event.
2. Identify affected users, hosts, IP addresses, and services.
3. Review surrounding events in the same time window.
4. Correlate failures, successes, and follow-on activity.
5. Determine whether activity is malicious, benign, or requires escalation.
6. Record evidence, rationale, and next steps.

## Example Splunk patterns

```spl
index=*
| stats count by sourcetype
```

```spl
index=auth action=failure
| stats count by user, src_ip
| sort - count
```

## Skills demonstrated

Splunk search, Wazuh monitoring, alert triage, authentication-event analysis, SSH brute-force investigation, event correlation, incident documentation, and detection-engineering fundamentals.

> This document excludes challenge answers, credentials, flags, and assessment values.
