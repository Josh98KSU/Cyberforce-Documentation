# F014 — Second Root SSH Key (root@ip-10-0-32-83)

- **Severity:** CRITICAL
- **VM:** Task Server (OpenSUSE)
- **Status:** RESOLVED
- **Discovered:** 2026-11-13
- **Resolved:** 2026-11-13

## Summary
A second unauthorized SSH public key was found in `/root/.ssh/authorized_keys`, granting root access from a Red Team machine. The key comment identified the source as `root@ip-10-0-32-83`.

## Evidence
- File: `/root/.ssh/authorized_keys`
- Key type: `ssh-ed25519`
- Fingerprint comment: `root@ip-10-0-32-83`
- Full key: `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMiraGc9ARn+iDD3Iy8ozbqpc4GEaWE8JbiFRMmJlYvG root@ip-10-0-32-83`

## Impact
Direct root SSH access from the Red Team network (10.0.32.x). Combined with F010 (sshd config hijack) and F011 (blueteam key), the Red Team had multiple independent paths to root.

## Remediation
1. Backed up original file: `/root/.ssh/authorized_keys.bak-20261113-2348`
2. Removed the `root@ip-10-0-32-83` key
3. Retained the Control key (restricted, likely White Team — flagged for confirmation)

## Verification
- `cat /root/.ssh/authorized_keys` — only Control key remains
- No unauthorized root SSH logins in `journalctl -u sshd` post-remediation

## Notes
- Backup preserved at `/root/.ssh/authorized_keys.bak-20261113-2348`
- The retained Control key should be confirmed with White Team on Discord
- Related to F002 (omar.hassan key), F008 (Control key), F010 (sshd config), F011 (blueteam key)
