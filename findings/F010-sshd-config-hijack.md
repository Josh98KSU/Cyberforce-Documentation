# F010 — SSHd Config Drop-in + Hidden Keys Directory

**Discovered:** 2026-11-13 ~22:00 UTC (via F005 script analysis)
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** INVESTIGATE
**Related:** F005 (respawner), F011 (the actual key file)

## What It Is
File `/etc/ssh/sshd_config.d/99-cache.conf` added by `.cache-watch`.
Likely contains: `AuthorizedKeysFile /etc/ssh/.keys/%u .ssh/authorized_keys`
(Overrides normal SSH to look at `/etc/ssh/.keys/<username>` FIRST, where attacker places keys.)

This bypasses:
- Removing `~/.ssh/authorized_keys`
- Normal user access controls
- Any sanity check you do on home directories

Made immutable with `chattr +i`.

## Why Malicious
- Config is in a dotdir sibling naming scheme designed to blend in with normal package configs
- 99- prefix ensures it loads LAST and overrides default AuthorizedKeysFile
- Points to a hidden directory (`.keys`)
- Installed by a hidden daemon

## Investigation Needed
```
sudo cat /etc/ssh/sshd_config.d/99-cache.conf
sudo ls -la /etc/ssh/.keys/
sudo cat /etc/ssh/.keys/*
```

## Remediation Plan
1. **Read file first** to confirm what it does
2. **Fix F005 FIRST**
3. **Remove immutable flag:** `sudo chattr -i /etc/ssh/sshd_config.d/99-cache.conf`
4. **Delete config:** `sudo rm /etc/ssh/sshd_config.d/99-cache.conf`
5. **Remove the keys directory:**
   ```
   sudo chattr -R -i /etc/ssh/.keys/ 2>/dev/null
   sudo rm -rf /etc/ssh/.keys/
   ```
6. **Validate sshd config:** `sudo sshd -t`
7. **Reload sshd:** `sudo systemctl reload sshd`
8. **Verify from a new session you can still log in!**

## Remediation Actions Taken
- [ ] Not yet started
