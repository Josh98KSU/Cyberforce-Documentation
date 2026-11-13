# F011 — Hidden SSH AuthorizedKey for blueteam

**Discovered:** 2026-11-13 ~22:00 UTC (via F005 script analysis)
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** INVESTIGATE
**Related:** F005, F010

## What It Is
File `/etc/ssh/.keys/blueteam` — an SSH public key maintained by `.cache-watch`.
Works in conjunction with F010 (sshd_config override) to let Red Team SSH as `blueteam` (full sudo NOPASSWD).
Immutable via chattr +i.

Source payload: `/root/.cfc-baseline/payloads/authkey`

## Why Dangerous
blueteam has `(ALL : ALL) NOPASSWD: ALL` sudo permissions — this key grants root-equivalent access without even needing to know blueteam's password.

## Investigation Needed
```
sudo cat /etc/ssh/.keys/blueteam
sudo cat /root/.cfc-baseline/payloads/authkey
```

## Remediation Plan
Covered under F010 remediation (remove the entire /etc/ssh/.keys/ directory).

Additionally:
1. Change blueteam's password (if password auth is enabled)
2. Consider generating a new SSH key pair for yourself and replacing any legit authorized_keys

## Remediation Actions Taken
- [ ] Not yet started
