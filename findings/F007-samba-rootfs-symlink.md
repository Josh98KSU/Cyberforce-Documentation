# F007 — Samba rootfs Symlink + Wide Links Escape

**Discovered:** 2026-11-13 ~21:25 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** HIGH
**Status:** ✅ RESOLVED (2026-11-13 ~21:50 UTC)

## What It Was
`/srv/samba/data/rootfs` was a symlink to `/` (filesystem root).
Combined with smb.conf settings:
- `allow insecure wide links = yes`
- `follow symlinks yes` (syntax broken but intent clear)
- `wide links = yes` (on [data] share)
- `force user = root`
- `guest ok = yes`
- `create mask = 0777` / `directory mask = 0777`

This allowed **any unauthenticated user on the network to read and write the entire filesystem as root via SMB.**

## Remediation Actions Taken
- 2026-11-13 ~21:47: Removed `/srv/samba/data/rootfs` symlink (`sudo rm`)
- 2026-11-13 ~21:48: Backed up smb.conf to `/etc/samba/smb.conf.bak-2026-11-13`
- 2026-11-13 ~21:49: Rewrote smb.conf:
  - Removed `allow insecure wide links`, `force user = root`, `wide links = yes`, `follow symlinks yes`, `map to guest = Bad User`
  - Added `server min protocol = SMB2`, `client min protocol = SMB2`
  - Changed create mask to 0644
- 2026-11-13 ~21:50: `sudo testparm` validated
- 2026-11-13 ~21:50: `sudo systemctl restart smb` completed
- 2026-11-13 ~21:50: `smbclient -L localhost -N` confirms shares still visible

## Verification Needed
- [ ] Confirm scoring engine still accesses `score` share successfully after scoring interval
- [ ] Monitor for symlink re-appearing (would indicate another backdoor)
