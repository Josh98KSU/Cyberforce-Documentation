# F012 — Rogue systemd unit launching fake json_pp

- **ID:** F012
- **Severity:** CRITICAL
- **VM:** opensuse (Task Server)
- **Discovered:** 2026-11-13
- **Status:** INVESTIGATE

## Summary
`/etc/systemd/system/network-bridge.service` is the launcher that starts
`/bin/json_pp-3.13` (the fake binary from F004) at boot as root.

## Evidence
grep match: /etc/systemd/system/network-bridge.service:7:ExecStart=/bin/json_pp-3.13

Named "network-bridge" to look legitimate.

## Impact
Root-level persistence on every boot. F004 process respawns automatically
unless this unit is disabled and masked.

## Related
F004 (the binary this launches)

## Remediation
Part of consolidated kill order (pending).
