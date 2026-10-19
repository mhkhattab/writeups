# Network Traffic & Protocol Analysis — LetsDefend Case Study

**Environment:** Authorized LetsDefend labs  
**Tools:** Wireshark, tcpdump  
**Focus:** PCAP analysis, SNMP, DNS, authentication/brute-force traffic, protocol evidence

## What I worked on

- Analyzed packet captures with Wireshark and tcpdump.
- Identified source/destination systems, protocol versions, service ports, and request/response patterns.
- Investigated SNMP traffic to determine management targets, protocol behavior, and environment details from packet evidence.
- Analyzed brute-force traffic to identify the protocol being targeted, target account context, target host, and repeated authentication attempts.
- Used packet counts, stream reconstruction, and protocol fields to support conclusions.

## Investigation workflow

1. Establish capture scope and identify dominant protocols.
2. Review endpoints and conversations.
3. Filter by protocol, host, port, or event type.
4. Reconstruct relevant request/response sequences.
5. Count repeated attempts when frequency matters.
6. Correlate packet evidence with the investigation question.
7. Document the technical conclusion and defensive significance.

## Useful techniques

```bash
tcpdump -nn -r capture.pcap
```

Wireshark techniques used included display filters, Conversations/Endpoints, Follow TCP Stream, protocol hierarchy, packet-field inspection, DNS review, source/destination filtering, and request counting.

## Defensive takeaways

- Restrict and monitor management protocols such as SNMP.
- Correlate repeated authentication attempts with account, source, time window, and any successful logins.
- Verify directionality before assigning attacker and target roles.
- Use packet captures to validate or deepen findings from higher-level logs.

## Skills demonstrated

Wireshark, tcpdump, PCAP triage, SNMP analysis, DNS analysis, authentication/brute-force traffic analysis, protocol identification, and evidence-based documentation.

> This document excludes challenge answers, credentials, flags, and assessment values.
