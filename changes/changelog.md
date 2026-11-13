# Change Log — All VMs

Format: `YYYY-MM-DD HH:MM UTC | VM | [CATEGORY] action | reason/finding-id | result`

Categories: SNAPSHOT, INVESTIGATE, REMEDIATE, HARDEN, CONFIG, USER, SERVICE, FIREWALL, ROLLBACK

---

## 2026-11-13

21:00 UTC | opensuse | [INVESTIGATE] Logged in, ran recon commands | initial assessment | discovered 8 findings (F001-F008)

21:15 UTC | opensuse | [SNAPSHOT] Created AWS AMI `opensuse-pre-remediation-2026-11-13` | baseline before remediation | success

21:30 UTC | opensuse | [REMEDIATE] Removed /srv/samba/data/rootfs symlink | F007 — anonymous write-anywhere-as-root | rm success, verified gone

21:32 UTC | opensuse | [CONFIG] Edited /etc/samba/smb.conf — removed `allow insecure wide links`, `force user = root`, `wide links = yes` on [data] | F007 | testparm OK, smb restarted, SMB still listening on 139/445

21:33 UTC | opensuse | [VERIFY] Ran `smbclient -L localhost -N` — shares still visible | post-samba-change verify | PASS

2026-11-13 ~21:50 UTC | opensuse | [REMEDIATE] Removed /srv/samba/data/rootfs symlink + hardened smb.conf | F007 | shares still accessible, SMB2 only

---
