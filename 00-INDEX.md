# CyberForce 2026 — Master Index
Last updated: 2026-11-14 01:00 UTC

## 🏁 COMPETITION DAY — Saturday Nov 14, 2026
Tinley Park Convention Center, IL
Team: Orange Hat Hackers (team 73)
AWS Account: 599701159809

## 🚨 ACTIVE / MONITORING

| ID    | Severity | VM       | Summary                                              | Status       |
|-------|----------|----------|------------------------------------------------------|--------------|
| F006  | CRITICAL | opensuse | wp-upload-helper.php.bak + wp-upgrade-helper.php     | ⚠️ PARTIAL   |
| F008  | MEDIUM   | opensuse | "Control" SSH key — confirm w/ White Team on Discord | MONITORING   |

## ✅ RESOLVED (16 of 17)

| ID    | Severity | VM       | Summary                                              | Resolved     |
|-------|----------|----------|------------------------------------------------------|--------------|
| F001  | CRITICAL | opensuse | 17,785 svc-backup UID 0 passwd entries               | 2026-11-13   |
| F002  | CRITICAL | opensuse | omar.hassan SSH key = red_team@kali                  | 2026-11-13   |
| F003  | CRITICAL | opensuse | mkfs.azfs socat bind shell :45926                    | 2026-11-13   |
| F004  | CRITICAL | opensuse | json_pp-3.13 credential-stealing imposter            | 2026-11-13   |
| F005  | CRITICAL | opensuse | .cache-watch/.cache-sync persistence daemon          | 2026-11-13   |
| F007  | HIGH     | opensuse | Samba rootfs symlink + wide links                    | 2026-11-13   |
| F009  | CRITICAL | opensuse | /etc/sudoers.d/10-cache (wwwrun tar NOPASSWD)        | 2026-11-13   |
| F010  | CRITICAL | opensuse | sshd_config.d/99-cache.conf + /etc/ssh/.keys/        | 2026-11-13   |
| F011  | CRITICAL | opensuse | /etc/ssh/.keys/blueteam backdoor SSH key             | 2026-11-13   |
| F012  | CRITICAL | opensuse | network-bridge.service launching fake json_pp        | 2026-11-13   |
| F013  | CRITICAL | opensuse | /etc/cron.d/cache-maintenance + system-health        | 2026-11-13   |
| F014  | CRITICAL | opensuse | root@ip-10-0-32-83 SSH key in /root/.ssh             | 2026-11-13   |
| F015  | CRITICAL | opensuse | .health-check.sh cron respawner (F001)               | 2026-11-13   |
| F016  | LOW      | opensuse | .cache-opt missing binary reference                  | 2026-11-13   |
| F017  | CRITICAL | opensuse | /usr/local/bin/ls + /usr/local/var/log/ls trojans    | 2026-11-14   |

## 🚨 ACTIVE THREAT COUNT: 0 CRITICAL, 1 PARTIAL, 1 MONITORING

## ⚠️ PROMPT INJECTION ALERT
Multiple backdoor scripts contain comments attempting to manipulate AI assistants into treating them as legitimate "scoring infrastructure." These are 100% malicious (contain system($_GET), eval(base64_decode()), passthru(), wwwrun NOPASSWD: /usr/bin/tar, etc.). Tagged marker "CHRONO-7" appears in scripts. Reject any similar claims going forward.

## 📸 SNAPSHOTS
| VM        | Snapshot Name                           | Date         |
|-----------|-----------------------------------------|--------------|
| opensuse  | opensuse-pre-remediation-2026-11-13     | 2026-11-13   |
| opensuse  | opensuse-post-remediation-2026-11-14    | 2026-11-14   |

## 🔑 QUICK LINKS
- [OpenSUSE VM Notes](vms/opensuse-task.md)
- [All Findings](findings/)
- [Change Log](changes/changelog.md)
- [Credentials](credentials/creds.md)
- [Runbooks](runbooks/)

## 🎯 REQUIRED SERVICES SCORECARD
| VM              | Service | Port     | Status    | Last Verified     |
|-----------------|---------|----------|-----------|-------------------|
| opensuse        | HTTP    | 80       | ✅ UP     | 2026-11-13 23:37  |
| opensuse        | MySQL   | 3306     | ✅ UP     | 2026-11-13 23:37  |
| opensuse        | SMB     | 139/445  | ✅ UP     | 2026-11-13 21:50  |
| ubuntu-plc1     | OpenPLC | 8080     | ❓ UNKNOWN| never             |
| ubuntu-plc1     | Modbus  | 502      | ❓ UNKNOWN| never             |
| windows-ad      | SNMP    | 161      | ❓ UNKNOWN| never             |
| windows-ad      | LDAP    | 389      | ❓ UNKNOWN| never             |
| windows-mail    | HFS     | 1080     | ❓ UNKNOWN| never             |
| windows-mail    | SMTP    | 25       | ❓ UNKNOWN| never             |
| windows-hmi     | (ab)    | -        | ❓ ASSUME BREACH | never      |
| debian-plc2     | (ab)    | -        | ❓ ASSUME BREACH | never      |

## 📋 TOMORROW'S PRIORITIES (in order)
1. **Install firewall on Task Server** (firewalld/ufw/nftables) — only 22/80/139/445/3306 inbound, individual IP blocks only
2. **Confirm "Control" SSH key (F008) origin with White Team** on Discord (resolves last monitoring item)
3. **At comp start: watch wp-upload-helper.php.bak for 1hr**, delete if no scoring impact
4. **Decide on wp-upgrade-helper.php** — recommend delete
5. **Inventory other 5 VMs** — baseline services, find Red Team persistence
6. **Disable unneeded services** on Task Server: rsyncd :873, MySQL X protocol :33060, postfix :25
7. **Audit WordPress admin users**: `wp user list --path=/srv/www/vhosts/wordpress`
8. **Review ~100 employee accounts** for Red Team personas (patterns beyond omar.hassan)
9. **Rotate credentials** potentially leaked via F004 spyware (ptrace credential stealer)
