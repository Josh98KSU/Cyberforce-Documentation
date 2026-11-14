# F008 — Unknown "Control" SSH Key + Legacy authorized_keys2

**Discovered:** 2026-11-13 ~21:30 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** MEDIUM
**Status:** ✅ RESOLVED (with monitoring flag)

## What It Is
An SSH key labeled "Control" is present in:
- `/root/.ssh/authorized_keys` — wrapped with `command="echo 'Please login as ec2-user'..."` restriction
- `/home/ec2-user/.ssh/authorized_keys` — no restrictions (direct login as ec2-user)
- `/root/.ssh/authorized_keys2` existed with a different key labeled "vault"

Key fingerprint starts with `AAAAB3NzaC1yc2EAAAADAQABAAABAQC/vdVWwVTi...` labeled `Control`.

/root/.ssh/authorized_keys2 contents (now deleted):
`ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPcNkZyTACb/dQmA8JxTcJVZqckOZK1iE9+v6JGjcUYE vault`

authorized_keys2 is a deprecated OpenSSH location but still honored by default in sshd on many distros — direct root SSH access via this key.

## Analysis
The "Control" key naming is ambiguous — could be White Team competition infrastructure OR Red Team disguise. Decision made to retain it (with its forced-command restriction on root) rather than risk disqualifying ourselves by breaking White Team access. The "vault" key in authorized_keys2 had no restriction and no legitimate explanation → removed.

## Remediation Actions Taken
- 2026-11-13 ~23:46: **DELETED** `/root/.ssh/authorized_keys2` (vault key, legacy location, no restriction)
- 2026-11-13 ~23:46: **RETAINED** Control key in `/root/.ssh/authorized_keys` (forced-command restricted)
- 2026-11-13 ~23:46: **RETAINED** Control key in `/home/ec2-user/.ssh/authorized_keys` (ec2-user is locked AWS standard account)
- Monitor auth logs for Control key usage patterns

## Remaining Action
- [ ] Confirm "Control" key origin with White Team via Discord help desk (ASAP on competition day)
- [ ] If Red Team origin confirmed: remove from both locations and re-snapshot
