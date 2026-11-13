# OpenSUSE Task Server

## Basics
- **IP:** 10.0.172.145
- **Hostname:** ip-10-0-172-145
- **OS:** openSUSE Leap 16.0
- **Kernel:** 6.12.0-160000.35-default
- **My user:** blueteam (uid 1004, wheel group, NOPASSWD sudo)
- **Disk:** 30GB, 21% used
- **RAM:** 3.7GB

## Required Services (SCORED)
| Service | Port     | Daemon          | Config Path                          |
|---------|----------|-----------------|--------------------------------------|
| HTTP    | 80       | apache2         | /etc/apache2/                        |
| MySQL   | 3306     | mysql84         | /etc/my.cnf, /opt/mysql/             |
| SMB     | 139/445  | smbd            | /etc/samba/smb.conf                  |

## Web Application
- **App:** WordPress
- **DocRoot:** /srv/www/vhosts/wordpress/
- **Config:** /srv/www/vhosts/wordpress/wp-config.php
- **Also exists:** /srv/www/htdocs/info.php (phpinfo page — probably should remove)
- **Custom plugin:** cfc-historian (ICS-themed, suspicious)
- **Custom theme:** vulcana-dynamics

## Samba Shares
| Share | Path             | Guest | Notes                              |
|-------|------------------|-------|------------------------------------|
| score | /srv/samba/score | YES   | Likely required by scoring engine  |
| data  | /srv/samba/data  | YES   | Had rootfs→/ symlink (REMOVED)     |

## Non-Required Services Running (Attack Surface)
- rsyncd :873 — not required, consider disabling
- MySQL X Protocol :33060 — not required, consider binding to localhost
- postfix :25 (localhost only) — fine

## User Accounts of Interest
- blueteam (me) — uid 1004
- ec2-user — AWS default
- score1, score2, patch — DO NOT MODIFY
- mysql — service account
- svc-backup — BACKDOOR (see F001)
- omar.hassan — BACKDOOR (see F002)
- ~100 "employee" accounts (C-Suite scenario)
- ~6 "old.*" former employee accounts (should disable per best practice)

## Snapshots Taken
| Date       | Name                                    | Reason               |
|------------|-----------------------------------------|----------------------|
| 2026-11-13 | opensuse-pre-remediation-2026-11-13     | Before remediation   |

## Active Findings on This VM
- F001 svc-backup x 17,785
- F002 omar.hassan SSH backdoor
- F003 mkfs.azfs socat bind shell
- F004 json_pp-3.13 imposter
- F005 .cache-watch/.cache-sync respawner
- F006 WordPress webshells x 3
- F007 Samba rootfs symlink
- F008 "Control" SSH key — unknown origin
