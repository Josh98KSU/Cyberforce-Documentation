# F004 — json_pp-3.13 Imposter Binary

**Discovered:** 2026-11-13 ~21:00 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED
**Related:** F012 (network-bridge.service was the launcher)

## What It Is
Fake binary at `/bin/json_pp-3.13`:
- Owner: `blueteam:blueteam` (should be root:root for /bin/)
- Size: 40,800 bytes (real json_pp is a 4,992-byte Perl wrapper)
- ELF executable, not stripped, with debug info
- Running as root since boot (PID 926) with child PID 934

Real `/bin/json_pp` also exists (4992 bytes, root:root) — legitimate.

## Why Malicious
- Perl's json_pp is a script wrapper, not an ELF binary
- Owned by non-root user but running as root = someone put it there via a privilege escalation
- Version "3.13" is not a json_pp version number (it's suspiciously similar to Python 3.13)
- Running continuously with child processes — json_pp is a one-shot JSON formatter
- Code analysis revealed: ptrace-attaches to passwd processes, netlink proc event listener, logged stolen credentials to `/home/blueteam/test.txt`

## Remediation Actions Taken
- 2026-11-13 ~23:42: Captured binary for evidence: `sudo cp /bin/json_pp-3.13 /root/ir-evidence-json_pp-3.13.bin`
- 2026-11-13 ~23:42: Killed PIDs: `sudo kill -9 926 934`
- 2026-11-13 ~23:42: Disabled launcher (F012): `sudo systemctl disable --now network-bridge.service`
- 2026-11-13 ~23:42: Removed binary: `sudo rm /bin/json_pp-3.13`
- 2026-11-13 ~23:43: Removed credential log: `sudo rm /home/blueteam/test.txt`
- 2026-11-13 ~23:43: Verified real json_pp still works: `json_pp --version`
- 2026-11-13 ~23:43: Monitored for respawn — none (F012 was the only launcher)
