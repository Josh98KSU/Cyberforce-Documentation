# F010 — SSHd Config Drop-in + Hidden Keys Directory

**Discovered:** 2026-11-13 ~22:00 UTC (via F005 script analysis)
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ✅ RESOLVED
**Related:** F005 (respawner — resolved), F011 (the key file this enabled)

## What It Is
File `/etc/ssh/sshd_config.d/99-cache.conf` added by `.cache-watch`.
Contained: `AuthorizedKeysFile /etc/ssh/.keys/%u .ssh/authorized_keys`
Overrode normal SSH to look at `/etc/ssh/.keys/<username>` FIRST, where attacker placed keys.

This bypassed:
- Removing `~/.ssh/authorized_keys`
- Normal user access controls
- Any sanity check on home directories

Made immutable with `chattr +i`.

## Why Malicious
- Config was in a drop-in naming scheme designed to blend in with normal package configs
- 99- prefix ensured it loaded LAST and overrode default AuthorizedKeysFile
- Pointed to a hidden directory (`.keys`)
- Installed by a hidden daemon

## Remediation Actions Taken
- 2026-11-13 ~23:33: F005 respawner disabled FIRST (prerequisite)
- 2026-11-13 ~23:34: Removed immutable flag: `sudo chattr -i /etc/ssh/sshd_config.d/99-cache.conf`
- 2026-11-13 ~23:34: Deleted config: `sudo rm /etc/ssh/sshd_config.d/99-cache.conf`
- 2026-11-13 ~23:35: Removed keys directory: `sudo chattr -R -i /etc/ssh/.keys/ && sudo rm -rf /etc/ssh/.keys/`
- 2026-11-13 ~23:35: Validated sshd config: `sudo sshd -t` → clean
- 2026-11-13 ~23:35: Reloaded sshd: `sudo systemctl reload sshd`
- 2026-11-13 ~23:35: Verified from new session — still able to log in as blueteam
