# F013 — Cron Persistence (cache-maintenance + system-health)

- **Severity:** CRITICAL
- **VM:** Task Server (OpenSUSE)
- **Status:** RESOLVED
- **Discovered:** 2026-11-13
- **Resolved:** 2026-11-13

## Summary
Two malicious cron drop-in files in `/etc/cron.d/` maintained Red Team persistence. `cache-maintenance` ran daily at 03:14 and `system-health` ran every 5 minutes, both re-establishing backdoor infrastructure if removed.

## Evidence
- `/etc/cron.d/cache-maintenance` — daily 03:14 trigger
- `/etc/cron.d/system-health` — every 5 minutes trigger
- Both referenced hidden scripts and binaries placed by the Red Team (related to F001, F003, F005, F015)

## Impact
Even if individual backdoors were cleaned, these cron jobs would respawn them within minutes (system-health) or overnight (cache-maintenance). This made manual cleanup ineffective without addressing the cron layer first.

## Remediation
1. Removed both cron files from `/etc/cron.d/`
2. Verified no other cron entries in `/etc/crontab`, `/var/spool/cron/`, or `/etc/cron.daily/` etc.
3. Confirmed no respawn after removal

## Verification
- `ls -la /etc/cron.d/` — only legitimate entries remain
- `crontab -l` for all users — clean
- Monitored for 10+ minutes post-cleanup, no backdoor respawn

## Notes
- Related to F001 (svc-backup respawn), F003 (socat respawn), F005 (cache-watch), F015 (.health-check.sh)
- The layered persistence (cron → script → binary → backdoor) is why cleanup had to follow a specific order
