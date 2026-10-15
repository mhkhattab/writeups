# TryHackMe — Dev Diaries

## Overview

**Platform:** TryHackMe  
**Category:** OSINT / Git Intelligence  
**Core techniques:** passive subdomain discovery, archive research, GitHub reconnaissance, commit metadata analysis, historical-file recovery

Dev Diaries demonstrates an important reconnaissance concept: infrastructure and source code can disappear from the live environment while traces survive in passive DNS, web archives, public Git repositories, and commit history.

Direct room answers, usernames, email addresses, and flags are intentionally omitted.

## Investigation Chain

```text
Primary domain
    ↓
Passive DNS / historical records
    ↓
Development or UAT hostname
    ↓
Archived page / developer attribution
    ↓
Public Git repository
    ↓
Commit metadata
    ↓
Historical source and deleted files
```

## 1. Active DNS Enumeration

A reasonable first attempt is active subdomain enumeration:

```bash
gobuster dns   --domain <DOMAIN>   -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

or:

```bash
subfinder -d <DOMAIN>
```

If current DNS enumeration produces little or nothing, that does not mean the hostname never existed.

Development systems often disappear before an investigation begins.

## 2. Passive DNS and Historical Search

Pivot to passive sources:

- certificate-transparency logs
- passive DNS databases
- WHOIS/history services
- search-engine indexes
- web archives

Queries:

```text
site:*.<DOMAIN>
"<DOMAIN>" "uat"
"<DOMAIN>" "staging"
"<DOMAIN>" "developer"
```

Certificate-transparency data is particularly useful because TLS certificates can expose hostnames long after the corresponding systems have been removed.

## 3. Wayback Machine

Once a historical development hostname is discovered, search archived captures:

```text
https://web.archive.org/
```

Archived pages may reveal:

- developer names
- footer attribution
- repository links
- email addresses
- technology fingerprints
- old API paths
- removed content

The key lesson is that deleting a host does not delete every copy of the site from third-party archives.

## 4. GitHub Reconnaissance

Search public GitHub content for:

- organization/domain name
- application name
- developer attribution
- unique strings recovered from archived HTML

Once a candidate repository is identified, clone it locally:

```bash
git clone <REPOSITORY_URL>
cd <REPOSITORY>
```

Inspect the complete commit graph:

```bash
git log --all --decorate --oneline
git log --all --format=fuller
```

A useful forensic format:

```bash
git log --all --format='%H|%an|%ae|%ad|%s'
```

This exposes:

- commit hash
- author
- author email
- timestamp
- commit message

## 5. Recovering Deleted Content

Git tracks history, not just the latest filesystem state.

List changed/deleted paths:

```bash
git log --all --name-status
```

Inspect a historical commit:

```bash
git show <COMMIT_SHA>
```

Recover a specific file from a historical commit:

```bash
git show <COMMIT_SHA>:<PATH>
```

Search every reachable revision for a pattern:

```bash
git rev-list --all
git grep -n '<PATTERN>' $(git rev-list --all)
```

This is why simply deleting a file in a later commit does not remove sensitive content from repository history.

## 6. Commit Messages as OSINT Evidence

Commit messages often reveal operational context:

- abandoned projects
- customer disputes
- deployment incidents
- emergency fixes
- internal names
- temporary credentials
- migration details

Treat commit messages as part of the evidence set, not just developer notes.

## 7. Developer Identity Leakage

Public commit metadata can reveal a personal email configured through:

```bash
git config --global user.name
git config --global user.email
```

Defensive options include:

- organization-managed identities
- GitHub-provided noreply addresses
- pre-publication review
- repository history rewriting when sensitive metadata must be removed

## 8. What "Delete" Means in Git

A Git commit points to a tree; trees point to blobs.

If a file existed in commit A and was deleted in commit B, the blob for commit A is still reachable through history unless history itself is rewritten and garbage-collected.

Conceptually:

```text
Commit A ──> Tree ──> secret.txt blob
   │
   └── Commit B ──> Tree without secret.txt
```

The file disappears from the latest checkout but remains recoverable from commit A.

## Defensive Recommendations

- keep private source repositories private
- use secret scanning
- use protected developer identities
- avoid putting credentials in commit history
- review commit messages before publishing
- remove `.git` from deployment artifacts
- rewrite history if genuinely sensitive data was committed
- rotate secrets even after history rewriting

## Lessons Learned

Dev Diaries is fundamentally a lesson in **persistence of development traces**:

1. DNS history can outlive hosts.
2. web archives can outlive deployments.
3. Git history can outlive deleted files.
4. commit metadata can outlive developer involvement.
5. OSINT becomes powerful when these sources are correlated.

## References

- TryHackMe Dev Diaries room: https://tryhackme.com/room/devdiaries
