# TryHackMe — Beach Bar

## Overview

**Platform:** TryHackMe  
**Category:** Boot2Root / Web Security / Linux Privilege Escalation  
**Difficulty:** Easy  
**Core techniques:** source-code review, credential exposure, unsafe YAML deserialization, command execution, Linux process inspection, credential leakage through process arguments

Beach Bar is a compact example of vulnerability chaining. A relatively small web-development mistake exposes access to an authenticated feature, the feature processes untrusted YAML unsafely, and a local secret-handling mistake enables privilege escalation.

The important lesson is not any single payload. It is how several trust-boundary failures combine into full compromise.

## Attack Chain

```text
Client-side information disclosure
        ↓
Authenticated jukebox access
        ↓
Playlist import/export analysis
        ↓
Unsafe YAML deserialization
        ↓
Command execution as low-privilege service user
        ↓
Linux process enumeration
        ↓
Privileged credential exposed in process arguments
        ↓
Privilege escalation
```

## 1. Web Enumeration

Start with normal application reconnaissance before using automated scanners.

Useful checks include:

```bash
curl -i http://<TARGET_IP>/
curl -s http://<TARGET_IP>/ | less
```

Inspect:

- HTML comments
- JavaScript files
- hidden form fields
- API endpoints
- authentication flows
- import/export functionality
- browser network requests

The application leaks authentication material in client-delivered source. I intentionally omit the credential value from this public repository.

### Security implication

Anything returned to the browser is visible to the user. HTML comments, JavaScript variables, hidden elements, and bundled frontend configuration must never be treated as secret storage.

## 2. Authenticated Feature Analysis

After obtaining legitimate lab access to the jukebox interface, the most interesting functionality is playlist import/export.

Exporting a normal playlist reveals a YAML-like serialization format. This immediately raises several questions:

1. Is the imported document parsed server-side?
2. Is it treated as plain data or as an object graph?
3. Does the parser support language-specific object tags?
4. Is schema validation performed before deserialization?

A harmless first test is preferable to jumping directly to a shell. The goal is to determine whether user-controlled YAML can invoke behavior outside the intended data model.

## 3. Unsafe YAML Deserialization

Python YAML libraries can become dangerous when an unsafe loader accepts language-specific object constructors.

Conceptually, unsafe code looks like:

```python
yaml.load(user_supplied_data, Loader=yaml.Loader)
```

A safer pattern is:

```python
yaml.safe_load(user_supplied_data)
```

or, better, a strict application-specific schema that allows only the fields and primitive types required by the playlist format.

A controlled proof-of-concept in an authorized lab can execute a harmless identity command through a Python-object tag. The objective is to prove that deserialization is crossing the data/code boundary.

Once server-side command execution is confirmed, a reverse shell can be established to the AttackBox or VPN host.

Listener:

```bash
nc -lvnp 4444
```

In a lab, the payload should be kept minimal and targeted only at the assigned machine.

## 4. Post-Exploitation Enumeration

After obtaining the low-privileged shell, enumerate the system methodically rather than relying on random privilege-escalation checks.

```bash
id
whoami
uname -a
hostname
sudo -l
ps aux
ss -lntup
find / -perm -4000 -type f 2>/dev/null
```

Also inspect:

- service processes
- writable scripts
- scheduled tasks
- environment variables
- configuration files
- command-line arguments
- open localhost services

The key weakness in this room is a privileged Python service whose command line exposes sensitive authentication material.

Useful inspection:

```bash
ps auxww
cat /proc/<PID>/cmdline | tr '\0' ' '
```

## 5. Why Process Arguments Are Dangerous for Secrets

Arguments passed to a process can be visible through:

- `ps`
- `/proc/<PID>/cmdline`
- monitoring agents
- debugging tools
- crash reports
- shell history, depending on launch method

This makes patterns such as the following risky:

```bash
python service.py --password <secret>
```

Preferred approaches include:

- a dedicated secret manager
- a protected configuration file with strict permissions
- a file descriptor / secret injection mechanism
- short-lived credentials
- avoiding reusable privileged passwords entirely

## 6. Technical Findings

### Finding 1 — Sensitive information in client-side source

**Impact:** unauthorized access to functionality that should require a trusted credential.

**Mitigation:**

- remove test/demo credentials before deployment
- rotate any leaked secret
- add secret scanning to CI/CD
- treat frontend source as public

### Finding 2 — Unsafe deserialization

**Impact:** attacker-controlled data becomes server-side code execution.

**Mitigation:**

- use safe loaders
- enforce an allowlisted schema
- reject unexpected YAML tags
- isolate parsing from privileged processes
- avoid deserializing language-native objects from untrusted input

### Finding 3 — Credential leakage through argv

**Impact:** local users can recover privileged credentials and escalate.

**Mitigation:**

- never pass reusable secrets in command-line arguments
- use secret stores or protected runtime injection
- rotate compromised credentials
- run services with the least privilege required

## 7. Detection Opportunities

Potential telemetry for this attack chain includes:

- unusual playlist-import requests
- YAML object tags in request bodies
- web-service child processes spawning shells or utilities
- outbound connections from the web-service account
- unexpected reads of `/proc/*/cmdline`
- authentication to a privileged account from a service user context

## Lessons Learned

Beach Bar demonstrates how a complete compromise can emerge from three ordinary mistakes:

1. a secret exposed to the browser;
2. unsafe parsing of untrusted data;
3. poor privileged-secret handling.

The room is a useful reminder that application security and operating-system security are not separate layers: flaws at both levels frequently chain together.

## References

- TryHackMe Beach Bar room: https://tryhackme.com/room/hh-beachbar-d849f7f7
