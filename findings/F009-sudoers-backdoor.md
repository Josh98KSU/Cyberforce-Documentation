# F009 — Sudoers Drop-in Backdoor (wwwrun tar NOPASSWD)

**Discovered:** 2026-11-13 ~22:00 UTC (via F005 script analysis)
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED
**Related:** F005 (respawner — resolved), F006 (webshells gave wwwrun code execution)

## What It Is
File `/etc/sudoers.d/10-cache` installed and maintained by `.cache-watch`.
Mode 0440 (readable by root only), made immutable with `chattr +i`.
Source payload: `/root/.cfc-baseline/payloads/10-cache`

Contents: `wwwrun ALL=(root) NOPASSWD: /usr/bin/tar`

## Why Malicious
`.cache-watch` installed it every 2 seconds if removed — real admins don't do this.
Exploit: wwwrun (Apache user) can run tar as root with no password.
tar has multiple root-escalation vectors:
```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```
Combined with any of the 3 webshells (F006) = instant root via HTTP.

## Remediation Actions Taken
- 2026-11-13 ~23:33: F005 respawner disabled FIRST (prerequisite)
- 2026-11-13 ~23:34: Removed immutable flag: `sudo chattr -i /etc/sudoers.d/10-cache`
- 2026-11-13 ~23:34: Deleted file: `sudo rm /etc/sudoers.d/10-cache`
- 2026-11-13 ~23:34: Verified sudoers still valid: `sudo visudo -c` → clean
- 2026-11-13 ~23:34: Verified gone: `ls /etc/sudoers.d/` shows no 10-cache
- Monitored 10+ minutes — no respawn
