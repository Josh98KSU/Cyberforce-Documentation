# CyberForce 2026 — Master Index
Last updated: 2026-11-13 21:30 UTC

## 🚨 ACTIVE THREATS (Not Yet Remediated)
| ID    | Severity | VM           | Summary                              | Status      |
|-------|----------|--------------|--------------------------------------|-------------|
| F001  | CRITICAL | opensuse     | 17,785 svc-backup UID 0 entries      | INVESTIGATE |
| F002  | CRITICAL | opensuse     | omar.hassan SSH key = red_team@kali  | INVESTIGATE |
| F003  | CRITICAL | opensuse     | mkfs.azfs socat bind shell :45926    | INVESTIGATE |
| F004  | CRITICAL | opensuse     | json_pp-3.13 imposter binary         | INVESTIGATE |
| F005  | CRITICAL | opensuse     | .cache-watch/.cache-sync respawner   | INVESTIGATE |
| F006  | CRITICAL | opensuse     | 3x WordPress webshells               | INVESTIGATE |
| F007  | HIGH     | opensuse     | Samba rootfs symlink + wide links    | INVESTIGATE |
| F008  | MEDIUM   | opensuse     | "Control" SSH key — legit or RT?     | INVESTIGATE |

## ✅ RECENTLY RESOLVED (Last 24h)
- (none yet)

## 🔑 QUICK LINKS
- [OpenSUSE VM Notes](vms/opensuse-task.md)
- [All Findings](findings/)
- [Change Log](changes/changelog.md)
- [Credentials](credentials/creds.md)

## 🎯 REQUIRED SERVICES SCORECARD
| VM              | Service | Port     | Status    | Last Verified  |
|-----------------|---------|----------|-----------|----------------|
| opensuse        | HTTP    | 80       | ✅ UP     | 2026-11-13 20:45 |
| opensuse        | MySQL   | 3306     | ✅ UP     | 2026-11-13 20:45 |
| opensuse        | SMB     | 139/445  | ✅ UP     | 2026-11-13 20:45 |
| ubuntu-plc1     | OpenPLC | 8080     | ❓ UNKNOWN| never          |
| ubuntu-plc1     | Modbus  | 502      | ❓ UNKNOWN| never          |
| windows-ad      | SNMP    | 161      | ❓ UNKNOWN| never          |
| windows-ad      | LDAP    | 389      | ❓ UNKNOWN| never          |
| windows-mail    | HFS     | 1080     | ❓ UNKNOWN| never          |
| windows-mail    | SMTP    | 25       | ❓ UNKNOWN| never          |

## 📸 LATEST SNAPSHOTS
| VM        | Snapshot Name                        | Date       |
|-----------|--------------------------------------|------------|
| opensuse  | opensuse-pre-remediation-2026-11-13  | 2026-11-13 |
