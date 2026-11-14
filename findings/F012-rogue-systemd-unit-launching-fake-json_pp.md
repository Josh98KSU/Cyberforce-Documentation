# F012 — Rogue systemd Unit Launching Fake json_pp

**Discovered:** 2026-11-13 ~21:10 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED
**Related:** F004 (the imposter binary this launched — resolved)

## Summary
`/etc/systemd/system/network-bridge.service` was the launcher that started
`/bin/json_pp-3.13` (the fake binary from F004) at boot as root.

## Evidence
grep match: `/etc/systemd/system/network-bridge.service:7:ExecStart=/bin/json_pp-3.13`

Named "network-bridge" to look legitimate — there is no such thing as a system network-bridge unit by default on OpenSUSE.

## Impact
Root-level persistence on every boot. F004 process respawned automatically on reboot. No actual network bridge functionality.

## Remediation Actions Taken
- 2026-11-13 ~23:42: Stopped service: `sudo systemctl stop network-bridge.service`
- 2026-11-13 ~23:42: Disabled service: `sudo systemctl disable network-bridge.service`
- 2026-11-13 ~23:42: Removed unit file: `sudo rm /etc/systemd/system/network-bridge.service`
- 2026-11-13 ~23:42: Reloaded systemd: `sudo systemctl daemon-reload`
- 2026-11-13 ~23:43: Verified no other systemd units reference json_pp or similar imposters
- 2026-11-13 ~23:43: Verified no other `.service` files owned by non-root user
