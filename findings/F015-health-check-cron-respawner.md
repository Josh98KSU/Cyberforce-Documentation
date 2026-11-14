# F015 — .health-check.sh Cron Respawner (F001)

- **Severity:** CRITICAL
- **VM:** Task Server (OpenSUSE)
- **Status:** RESOLVED
- **Discovered:** 2026-11-13
- **Resolved:** 2026-11-13

## Summary
A hidden script at `/usr/local/bin/.health-check.sh` ran via cron every 5 minutes and re-injected the 17,785 `svc-backup` UID 0 entries into `/etc/passwd` (F001) if they had been cleaned. This was the primary respawn mechanism for the passwd pollution attack.

## Evidence
- Script: `/usr/local/bin/.health-check.sh` (hidden via dot-prefix)
- Cron trigger: every 5 minutes (via `/etc/cron.d/system-health`, see F013)
- Behavior: checked `/etc/passwd` for `svc-backup` entries, re-added them from an embedded template if missing

## Impact
Made F001 cleanup futile unless this respawner was found and disabled first. Every 5 minutes, the passwd file would be re-polluted with 17,785 fake root accounts.

## Remediation
1. Removed `/usr/local/bin/.health-check.sh`
2. Removed its cron trigger (F013)
3. Re-cleaned `/etc/passwd` of svc-backup entries
4. Verified no respawn over extended monitoring period

## Verification
- `ls -la /usr/local/bin/.health*` — no such file
- `grep svc-backup /etc/passwd | wc -l` — returns 0
- Monitored `/etc/passwd` for 15+ minutes, no re-injection

## Notes
- This is why F001 cleanup had to happen AFTER F013/F015 removal
- The `.health-check` naming was designed to blend in with legitimate system monitoring
- Related to F001 (svc-backup pollution), F013 (cron persistence)
