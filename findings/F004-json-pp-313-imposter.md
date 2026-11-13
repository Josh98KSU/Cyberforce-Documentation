# F004 — json_pp-3.13 Imposter Binary

**Discovered:** 2026-11-13 ~21:00 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** INVESTIGATE

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

## Open Questions
- What does it actually do? (C2 beacon? Keylogger? Another listener?)
- What systemd unit or script launches it?
- Where does /bin/json_pp-3.13 get re-copied from if we delete it?

## Investigation TODO
```
sudo strings /bin/json_pp-3.13 | head -100
sudo ls -la /proc/926/cwd
sudo cat /proc/926/environ | tr '\0' '\n'
sudo ss -tnp | grep 926
sudo lsof -p 926 2>/dev/null
sudo grep -rE 'json_pp' /etc/systemd/ /usr/lib/systemd/ /root/.cfc-baseline/ 2>/dev/null
```

## Remediation Plan
1. Investigate parent / launcher
2. Capture binary for evidence: `sudo cp /bin/json_pp-3.13 /root/ir-evidence-json_pp-3.13.bin`
3. Kill PIDs: `sudo kill -9 926 934`
4. Remove binary: `sudo rm /bin/json_pp-3.13`
5. Monitor for respawn

## Remediation Actions Taken
- [ ] Not yet started
