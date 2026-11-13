# F009 — Sudoers Drop-in Backdoor

**Discovered:** 2026-11-13 ~22:00 UTC (via F005 script analysis)
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** INVESTIGATE — NEED TO READ FILE
**Related:** F005 (respawner)

## What It Is
A file `/etc/sudoers.d/10-cache` is installed and maintained by `.cache-watch`.
Mode 0440 (readable by root only), made immutable with `chattr +i`.
Source payload: `/root/.cfc-baseline/payloads/10-cache`

## Why Malicious
`.cache-watch` installs it every 2 seconds if removed — real admins don't do this.
Based on naming and placement, almost certainly grants passwordless sudo to omar.hassan or similar attacker account.

## Investigation Needed
```
sudo cat /etc/sudoers.d/10-cache
sudo cat /root/.cfc-baseline/payloads/10-cache
```

## Remediation Plan
1. **Read it first** to know what it grants
2. **Fix F005 FIRST** (stop the respawner)
3. **Remove immutable flag:** `sudo chattr -i /etc/sudoers.d/10-cache`
4. **Delete file:** `sudo rm /etc/sudoers.d/10-cache`
5. **Verify sudoers still valid:** `sudo visudo -c`
6. **Verify:** `sudo cat /etc/sudoers.d/` contents

## Remediation Actions Taken
- [ ] Not yet started
