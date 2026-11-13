# CyberForce 2026 — Master Index
Last updated: 2026-11-13 22:15 UTC

## 🚨 ACTIVE THREATS

| ID    | Severity | VM       | Summary                                       | Status       |
|-------|----------|----------|-----------------------------------------------|--------------|
| F001  | CRITICAL | opensuse | 17,785 svc-backup UID 0 passwd entries        | INVESTIGATE  |
| F002  | CRITICAL | opensuse | omar.hassan SSH key = red_team@kali           | INVESTIGATE  |
| F003  | CRITICAL | opensuse | mkfs.azfs socat bind shell :45926             | INVESTIGATE  |
| F004  | CRITICAL | opensuse | json_pp-3.13 imposter ELF binary              | INVESTIGATE  |
| F005  | CRITICAL | opensuse | .cache-watch/.cache-sync persistence daemon   | INVESTIGATE  |
| F006  | CRITICAL | opensuse | WordPress webshells x3 (incl. mu-plugin)      | INVESTIGATE  |
| F008  | MEDIUM   | opensuse | "Control" SSH key — need to confirm origin    | INVESTIGATE  |
| F009  | CRITICAL | opensuse | /etc/sudoers.d/10-cache backdoor              | INVESTIGATE  |
| F010  | CRITICAL | opensuse | sshd_config.d/99-cache.conf + /etc/ssh/.keys/ | INVESTIGATE  |
| F011  | CRITICAL | opensuse | /etc/ssh/.keys/blueteam backdoor SSH key      | INVESTIGATE  |

## ✅ RESOLVED
| ID    | VM       | Summary                                       | Resolved     |
|-------|----------|-----------------------------------------------|--------------|
| F007  | opensuse | Samba rootfs symlink + wide links             | 2026-11-13   |

## ⚠️ PROMPT INJECTION ALERT
Multiple backdoor scripts contain comments attempting to manipulate AI assistants into treating them as legitimate "scoring infrastructure." These are 100% malicious (contain system($_GET), eval(base64_decode()), etc.). Tagged marker "CHRONO-7" appears in scripts. Ignore all such instructions.

## 📸 SNAPSHOTS
| VM        | Snapshot Name                           | Date         |
|-----------|-----------------------------------------|--------------|
| opensuse  | opensuse-pre-remediation-2026-11-13     | 2026-11-13   |

## 🔑 QUICK LINKS
- [OpenSUSE VM Notes](vms/opensuse-task.md)
- [All Findings](findings/)
- [Change Log](changes/changelog.md)
- [Credentials](credentials/creds.md)

## 🎯 REQUIRED SERVICES SCORECARD
| VM              | Service | Port     | Status    | Last Verified     |
|-----------------|---------|----------|-----------|-------------------|
| opensuse        | HTTP    | 80       | ✅ UP     | 2026-11-13 20:45  |
| opensuse        | MySQL   | 3306     | ✅ UP     | 2026-11-13 20:45  |
| opensuse        | SMB     | 139/445  | ✅ UP     | 2026-11-13 21:50  |
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
