# F011 — Hidden SSH AuthorizedKey for blueteam

**Discovered:** 2026-11-13 ~22:00 UTC (via F005 script analysis)
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED
**Related:** F005 (respawner — resolved), F010 (sshd config hijack — resolved)

## What It Is
File `/etc/ssh/.keys/blueteam` — an SSH public key maintained by `.cache-watch`.
Worked in conjunction with F010 (sshd_config override) to let Red Team SSH as `blueteam` (full sudo NOPASSWD).
Immutable via chattr +i.

Source payload: `/root/.cfc-baseline/payloads/authkey`

Key contents: `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFMzOvmY3c5jNd0Rt6OzcAeJqgyb8jqwxnnw8kEiXIH7 cfc-redteam`

## Why Dangerous
- Comment literally says "cfc-redteam" — no ambiguity about intent
- blueteam has `(ALL : ALL) NOPASSWD: ALL` sudo permissions
- This key granted root-equivalent access without even needing blueteam's password

## Remediation Actions Taken
- 2026-11-13 ~23:35: Removed as part of F010 cleanup (entire `/etc/ssh/.keys/` directory deleted)
- 2026-11-13 ~23:35: Verified no other copies anywhere on filesystem
- 2026-11-13 ~23:36: Confirmed blueteam SSH still works from laptop (own key intact in `/home/blueteam/.ssh/authorized_keys`)

## Remaining (Optional) Hardening
- [ ] Rotate blueteam SSH keypair (generate new keypair, replace authorized_keys entry)
- [ ] Change blueteam password (currently uses team-shared password)
- [ ] Consider disabling blueteam NOPASSWD sudo during competition (risky — may slow legitimate response)
