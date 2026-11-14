# F003 — mkfs.azfs Bind Shell on Port 45926

**Discovered:** 2026-11-13 ~20:55 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED
**Related:** F017 (ls trojan was the respawn mechanism — now resolved)

## What It Is
`/usr/bin/mkfs.azfs` was a symlink → `/usr/bin/socat1` (SUID root)
Running as: `mkfs.azfs TCP-LISTEN:45926,fork EXEC:/bin/bash`
Anyone connecting to port 45926 gets a root shell.

## Evidence
```
PID 206074, PPID 1, Uid: 1004 0 0 0 (ruid=blueteam, euid=root via SUID)
LISTEN 0.0.0.0:45926
/usr/bin/mkfs.azfs → socat1 (SUID root)
```

## Mystery (SOLVED)
Process ruid was 1004 (blueteam) but I didn't run it. PPID was 1 (systemd). Investigation of shell init files (.bashrc, .profile, /etc/bash.bashrc, /etc/profile.d/) found nothing.

**Root cause discovered 2026-11-14 ~00:30:** F017 — a bash trojan at `/usr/local/bin/ls` (and a second copy at `/usr/local/var/log/ls`) was respawning the socat backdoor every time ANY `ls` command ran. The trojan checked if port 45926 was open; if not, it launched `mkfs.azfs TCP-LISTEN:45926,fork EXEC:/bin/bash &`. This is why the backdoor kept coming back despite cleanup. The ruid=1004 came from our own `ls` invocations triggering the trojan.

## Remediation Actions Taken
- 2026-11-13 ~23:40: Killed socat process: `sudo kill -9 206074`
- 2026-11-13 ~23:40: Removed symlink: `sudo rm /usr/bin/mkfs.azfs`
- 2026-11-13 ~23:40: Stripped SUID from socat1: `sudo chmod u-s /usr/bin/socat1`
- 2026-11-13 ~23:41: Verified port closed: `sudo ss -tlnp | grep 45926` — nothing
- 2026-11-13 ~23:41: Reinstalled clean socat: `sudo zypper install --force socat`
- 2026-11-14 ~00:30: Discovered and removed F017 ls trojans (the actual respawn mechanism)
- 2026-11-14 ~00:49: Final verification — port 45926 not listening, only `/usr/bin/ls` exists
