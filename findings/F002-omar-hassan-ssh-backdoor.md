# F002 — omar.hassan SSH Backdoor Account

**Discovered:** 2026-11-13 ~20:50 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED

## What It Is
User `omar.hassan` (UID 1085) has an SSH authorized_keys entry with the comment `red_team@kali`.

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGDbq+DD6puDMNC3l4idJ6VIlsVq8GtgJsaOxFX5JcdM red_team@kali
```

## Why Malicious
- Key comment literally says "red_team@kali"
- Account blends in with the employee naming pattern (many `firstname.lastname` users exist)
- Empty `.bash_history` = never been used or was cleared

## Open Question (Resolved)
Is `omar.hassan` listed as a legitimate employee in the C-Suite scenario document? Decision: account locked rather than deleted to preserve evidence and avoid breaking any scenario dependencies.

## Remediation Actions Taken
- 2026-11-13 ~23:48: Backed up authorized_keys → `/home/omar.hassan/.ssh/authorized_keys.bak-20261113-2348`
- 2026-11-13 ~23:48: Removed Red Team SSH key from `/home/omar.hassan/.ssh/authorized_keys`
- 2026-11-13 ~23:48: Locked account: `sudo passwd -l omar.hassan`
- 2026-11-13 ~23:49: Killed any active sessions: `sudo pkill -KILL -u omar.hassan`
- 2026-11-13 ~23:49: Verified account locked: `sudo passwd -S omar.hassan` shows LK
- Home directory preserved for evidence (not deleted)
