# F008 — Unknown "Control" SSH Key

**Discovered:** 2026-11-13 ~21:30 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** MEDIUM (needs investigation)
**Status:** INVESTIGATE

## What It Is
An SSH key labeled "Control" is present in:
- `/root/.ssh/authorized_keys` — wrapped with `command="echo 'Please login as ec2-user'..."` restriction
- `/home/ec2-user/.ssh/authorized_keys` — no restrictions (direct login as ec2-user)
- `/root/.ssh/authorized_keys2` also exists (87 bytes) — legacy location, often forgotten

Key fingerprint starts with `AAAAB3NzaC1yc2EAAAADAQABAAABAQC/vdVWwVTi...` labeled `Control`.

## Open Question
Is "Control" the CyberForce competition control infrastructure (White Team SSH access for monitoring/scoring) OR is it Red Team?

Also: there's a second key `ssh-ed25519 ... root@ip-10-0-32-83` in root's authorized_keys.

## Why I'm Cautious
- Removing a legitimate control key could get you disqualified or cause scoring failures
- Keeping a Red Team key means they have persistent access
- The naming "Control" is deliberately ambiguous

## Investigation Plan
1. Check CyberForce documentation / Rules / Discord help desk for info on "Control" key
2. Check if other team members' machines have the same key (if so, likely legitimate)
3. Check /root/.ssh/authorized_keys2 contents — this is a VERY suspicious legacy location
4. Monitor auth logs for who connects with this key

## Remediation Plan
- Do NOT remove root's `authorized_keys` Control key (safer — it has a forced command restriction anyway)
- **DO remove `/root/.ssh/authorized_keys2`** (legacy location, often malicious use)
- **DO** investigate ec2-user's unrestricted Control key — probably safe to leave, but worth confirming

## Remediation Actions Taken
- [ ] Not yet started
