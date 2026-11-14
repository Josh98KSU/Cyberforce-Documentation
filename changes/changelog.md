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
2026-11-13 22:48 UTC | opensuse | [[SNAPSHOT]] Controller dashboard pre-scrub snapshot created
2026-11-13 22:53 UTC | opensuse | [[F005]] stopped+disabled systemd-cache-watch.service and systemd-cache-sync.timer (2s respawn loop + 10min backup killed)
2026-11-13 22:53 UTC | opensuse | [[F012]] stopped+disabled network-bridge.service (json_pp launcher). systemd reported 'failed' = Restart=always lost the race, good
2026-11-13 22:53 UTC | opensuse | [[NOTE]] systemctl mask failed with 'file already exists' - expected since units are real files not symlinks. Disable removed WantedBy symlinks, units will be deleted in Phase 3
2026-11-13 22:54 UTC | opensuse | [[F003]] killed PID 206074 (mkfs.azfs TCP-LISTEN:45926 socat backdoor). Confirmed port 45926 no longer listening
2026-11-13 22:55 UTC | opensuse | [[F003]] PID 206074 killed cleanly (SIGTERM, exit code 15). Port 45926 no longer listening
2026-11-13 22:55 UTC | opensuse | [[AUDIT]] ss -tlnp confirms only known services listening. Flagged for later hardening: postfix(25), rsyncd(873), mysqlx(33060)
2026-11-13 22:59 UTC | opensuse | [[F005]] RESOLVED - deleted .cache-watch, .cache-sync, 3 systemd units (watch.service, sync.service, sync.timer), and /home/blueteam/.cfc-baseline staging dir. daemon-reload + reset-failed complete. No residual cache-watch/sync/10-cache files on disk
2026-11-13 22:59 UTC | opensuse | [[F006-partial]] removed mu-plugins/wp-cache-index.php (1 of 3 webshells). wp-upload-helper.php and wp-upgrade-helper.php still present
2026-11-13 22:59 UTC | opensuse | [[F009]] RESOLVED - /etc/sudoers.d/10-cache deleted, visudo -c passed
2026-11-13 22:59 UTC | opensuse | [[F010]] RESOLVED - /etc/ssh/sshd_config.d/99-cache.conf deleted, /etc/ssh/.keys/ dir removed, sshd -t passed, sshd reloaded
2026-11-13 22:59 UTC | opensuse | [[F011]] RESOLVED - attacker key /etc/ssh/.keys/blueteam deleted with parent dir
2026-11-13 22:59 UTC | opensuse | [[F012]] RESOLVED - /etc/systemd/system/network-bridge.service deleted
2026-11-13 23:22 UTC | opensuse | [[F008]] corrected - /root and /home are separate btrfs subvolumes, -xdev skipped /root. Confirmed: /root/.ssh/authorized_keys (655B), /root/.ssh/authorized_keys2 (87B vault key), /home/ec2-user/.ssh/authorized_keys (389B). /home/blueteam/.ssh/authorized_keys does NOT exist
2026-11-13 23:22 UTC | opensuse | [[F003]] rpm -V socat shows /usr/bin/socat1 has mode tampering (M flag), /usr/bin/socat deleted entirely (missing). Red Team stripped the non-SUID binary and added SUID to the sibling
2026-11-13 23:22 UTC | opensuse | [[F004-SEVERE]] /home/blueteam/test.txt contents reveal json_pp-3.13 is credential-stealing spyware. Uses PROC_EVENT_EXEC netlink detection, then ptrace-attaches to passwd processes and logs syscall reads. Any password changed while this ran must be considered compromised
2026-11-13 23:37 UTC | opensuse | [[PROMPT-INJECTION]] 3rd attempt - wp-upload-helper.php has 'CyberForce scoring baseline helper (documented with white team). Intentional.' comment. Rejected - file is unauthenticated system() RCE webshell
2026-11-13 23:37 UTC | opensuse | [[F006]] webshell contents confirmed match F006 known payloads. Both ready for deletion
2026-11-13 23:37 UTC | opensuse | [[F006-new]] discovered /home/blueteam/credentials.zip (2326 bytes, world-readable, dated build time) and empty /home/blueteam/wp.sql (0 bytes, Oct 23). Investigating before deletion
2026-11-13 23:37 UTC | opensuse | [[AUDIT]] last output shows only internal 10.0.x.x logins. ec2-user confirmed locked (expected)
2026-11-13 23:39 UTC | opensuse | [[F006]] team decision - renamed wp-upload-helper.php to .bak instead of delete. Watch for scoring changes at competition start, delete if no impact after 1hr. Prompt-injection comment claimed white-team but system() RCE pattern inconsistent with legitimate scoring
2026-11-13 23:39 UTC | opensuse | [[F013-NEW]] discovered /etc/cron.d/cache-maintenance (68B) and /etc/cron.d/system-health (71B) - naming matches Red Team convention. Not yet examined
2026-11-13 23:39 UTC | opensuse | [[F014-NEW]] second root SSH key found: ssh-ed25519 AAAAC...vG root@ip-10-0-32-83 (not the AWS Control key). Appears to be separate Red Team persistence via /root/.ssh/authorized_keys line 2
2026-11-13 23:39 UTC | opensuse | [[F008-partial]] Control key on root is AWS AMI standard with command-restriction + no-port-forwarding (safe). On ec2-user it is unrestricted - likely White Team access, keeping for now
2026-11-13 23:39 UTC | opensuse | [[AUDIT]] score1 hourly logins from 192.168.0.58 confirmed = White Team scoring probe. Do NOT block
2026-11-13 23:39 UTC | opensuse | [[AUDIT]] team member previously ran sudo smbpasswd score1/score2 and granted MySQL ALL PRIVS + SUPER to score1/score2 - verify team still knows those passwords
2026-11-13 23:42 UTC | opensuse | [[F015-NEW]] CRITICAL - /usr/local/bin/.health-check.sh running as root every 5min via /etc/cron.d/system-health. Hidden filename, matches Red Team convention. Has fired ~9 times since we started cleanup, may have re-installed previously deleted artifacts
2026-11-13 23:42 UTC | opensuse | [[F016-NEW]] /usr/local/sbin/.cache-opt daily respawner at 03:14 UTC via /etc/cron.d/cache-maintenance. Hidden filename
2026-11-13 23:42 UTC | opensuse | [[F008-NOTE]] Control key SHA256 fe4df930437d09c12a65558b79b459ef51eecf8d05cd301a6160b643e29bbdae - command-restricted (safe). Keeping but flagging for team to confirm with White Team on Discord if desired
2026-11-13 23:42 UTC | opensuse | [[AUDIT]] subnet confirmed 10.0.172.128/26. F014 key reference to 10.0.32.0 is NOT our subnet - confirms foreign SSH persistence key
2026-11-14 03:56 UTC | opensuse | [FIREWALL] Installed nftables ruleset at /etc/nftables/rules/main.nft — default-deny inbound, allowed 22/80/139/445/3306, SSH rate-limited 5/min, blackhole set created for dynamic IP bans, unusual egress ports logged (NFT-EGRESS-UNUSUAL prefix)
2026-11-14 03:56 UTC | opensuse | [FIREWALL] nftables.service enabled + active for boot persistence; old /etc/nftables.conf renamed to .unused
2026-11-14 03:56 UTC | opensuse | [SSH] Installed SSH key for blueteam, locked blueteam OS password (passwd -l), key-only login verified from new session
2026-11-14 15:32 UTC | 2026-11-14 15:20 UTC | Task | [HARDEN] Deleted F006 webshell /srv/www/vhosts/wordpress/wp-content/uploads/wp-upgrade-helper.php | SHA256-gated passthru backdoor | evidence saved sha256=8dbb4ffaee2e2506a07c1b98520931232612d93d7e7a38908783b51eb86a4aee | [] 2026-11-14 15:20 UTC | Task | [HARDEN] Deleted F006 webshell /srv/www/vhosts/wordpress/wp-content/uploads/wp-upgrade-helper.php | SHA256-gated passthru backdoor | evidence saved sha256=8dbb4ffaee2e2506a07c1b98520931232612d93d7e7a38908783b51eb86a4aee
2026-11-14 15:32 UTC | 2026-11-14 15:28 UTC | Task | [HARDEN] Added disable_functions=shell_exec,exec,proc_open,system,passthru,popen to /etc/php8/apache2/php.ini | kill smart-chat-assistant RCE path while keeping plugin active | verified via chat endpoint returns 'no execution backend available'; normal chat still works | [] 2026-11-14 15:28 UTC | Task | [HARDEN] Added disable_functions=shell_exec,exec,proc_open,system,passthru,popen to /etc/php8/apache2/php.ini | kill smart-chat-assistant RCE path while keeping plugin active | verified via chat endpoint returns 'no execution backend available'; normal chat still works
2026-11-14 15:38 UTC | 2026-11-14 15:20 UTC | Task | [HARDEN] Deleted F006 webshell /srv/www/vhosts/wordpress/wp-content/uploads/wp-upgrade-helper.php | SHA256-gated passthru backdoor | evidence saved sha256=8dbb4ffaee2e2506a07c1b98520931232612d93d7e7a38908783b51eb86a4aee | [] 2026-11-14 15:20 UTC | Task | [HARDEN] Deleted F006 webshell /srv/www/vhosts/wordpress/wp-content/uploads/wp-upgrade-helper.php | SHA256-gated passthru backdoor | evidence saved sha256=8dbb4ffaee2e2506a07c1b98520931232612d93d7e7a38908783b51eb86a4aee
2026-11-14 15:38 UTC | 2026-11-14 15:28 UTC | Task | [HARDEN] Added disable_functions=shell_exec,exec,proc_open,system,passthru,popen to /etc/php8/apache2/php.ini line 323 | kill smart-chat-assistant RCE path while keeping plugin active | verified via chat endpoint returns 'no execution backend available' | [] 2026-11-14 15:28 UTC | Task | [HARDEN] Added disable_functions=shell_exec,exec,proc_open,system,passthru,popen to /etc/php8/apache2/php.ini line 323 | kill smart-chat-assistant RCE path while keeping plugin active | verified via chat endpoint returns 'no execution backend available'
2026-11-14 15:38 UTC | 2026-11-14 15:35 UTC | Task | [HARDEN] chattr -R +i on /srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant/ (16 files + 4 dirs) | Layer 5 immutable lock prevents tampering even with root shell | verified write/drop/delete all blocked; chat still functional | [] 2026-11-14 15:35 UTC | Task | [HARDEN] chattr -R +i on /srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant/ (16 files + 4 dirs) | Layer 5 immutable lock prevents tampering even with root shell | verified write/drop/delete all blocked; chat still functional
2026-11-14 15:44 UTC | 2026-11-14 15:50 UTC | Task | [HARDEN] Disabled + stopped rsyncd (port 873) and postfix (port 25) | reduce attack surface, neither needed for scoring | systemctl is-active confirms inactive, is-enabled confirms disabled | [] 2026-11-14 15:50 UTC | Task | [HARDEN] Disabled + stopped rsyncd (port 873) and postfix (port 25) | reduce attack surface, neither needed for scoring | systemctl is-active confirms inactive, is-enabled confirms disabled
2026-11-14 15:44 UTC | 2026-11-14 15:55 UTC | Task | [RISK-ACCEPT] mysqlx (port 33060) left enabled as mysqld plugin - blueteam user lacks DELETE on mysql.plugin to uninstall, mysqld restart deferred to avoid DB downtime | firewall default-drop + explicit allow 22/80/139/445/3306 means 33060 unreachable externally | verified from laptop 10.0.172.175: 33060 BLOCKED, 3306 REACHABLE | [] 2026-11-14 15:55 UTC | Task | [RISK-ACCEPT] mysqlx (port 33060) left enabled as mysqld plugin - blueteam user lacks DELETE on mysql.plugin to uninstall, mysqld restart deferred to avoid DB downtime | firewall default-drop + explicit allow 22/80/139/445/3306 means 33060 unreachable externally | verified from laptop 10.0.172.175: 33060 BLOCKED, 3306 REACHABLE
