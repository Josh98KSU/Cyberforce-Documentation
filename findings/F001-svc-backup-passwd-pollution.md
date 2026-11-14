# F001 — svc-backup UID 0 Account (×17,785)

**Discovered:** 2026-11-13 ~20:45 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED
**Related:** F005 (respawner), F013 (cron persistence), F015 (.health-check.sh) — all resolved

## What It Is
17,785 duplicate entries of `svc-backup` in `/etc/passwd`:
- UID 0, GID 0 (root-equivalent)
- Home directory: `/root`
- Password hash stored IN `/etc/passwd` (not shadow): `$1$xyz$XdpQe4qIkV8dua41Cgwa01` (MD5, crackable)
- Not in `/etc/shadow`

## Evidence
```
sudo grep -c "svc-backup" /etc/passwd  →  17785
sudo grep -n "svc-backup" /etc/passwd  →  starts at line 27
```

## Why Malicious
- Legitimate services use /etc/shadow for passwords
- No legitimate reason for 17,785 identical duplicate entries
- MD5 passwords were deprecated ~2010
- Home dir = /root means full access to root's files
- Pollution of /etc/passwd can also trigger integer overflows / enumeration DoS

## How I Found It
`cat /etc/passwd | grep -v nologin | grep -v /bin/false` during initial recon.

## Remediation Plan
1. First disable the respawner (F005/F013/F015) or new entries may reappear
2. Back up /etc/passwd: `sudo cp /etc/passwd /etc/passwd.bak`
3. Remove all svc-backup lines: `sudo sed -i '/^svc-backup:/d' /etc/passwd`
4. Verify: `sudo grep -c svc-backup /etc/passwd` should return 0
5. Verify /etc/passwd is still valid: `sudo pwck -r`
6. Verify svc-backup not in shadow: `sudo grep svc-backup /etc/shadow`
7. Verify no svc-backup processes running: `ps -ef | grep svc-backup`

## Remediation Actions Taken
- 2026-11-13 ~23:45: Disabled F015 (.health-check.sh) and F013 (cron persistence) FIRST to prevent respawn
- 2026-11-13 ~23:48: Backed up /etc/passwd → `/etc/passwd.bak-pre-f001-20261113-2348`
- 2026-11-13 ~23:48: Removed all svc-backup lines: `sudo sed -i '/^svc-backup:/d' /etc/passwd`
- 2026-11-13 ~23:49: Verified `grep -c svc-backup /etc/passwd` returns 0
- 2026-11-13 ~23:49: Verified `pwck -r` passes clean
- 2026-11-13 ~23:50: Confirmed svc-backup not in /etc/shadow
- 2026-11-13 ~23:50: Monitored for 15+ minutes — no respawn
