cyberforce-2026/
├── 00-INDEX.md                    ← One-page overview, your "homepage"
├── vms/
│   ├── opensuse-task.md           ← One file per VM
│   ├── windows-mail-file.md
│   ├── windows-ad.md
│   ├── ubuntu-plc1.md
│   ├── debian-plc2.md             ← Assume Breach — careful
│   └── windows-hmi.md             ← Assume Breach — careful
├── findings/
│   ├── F001-svc-backup-duplicates.md
│   ├── F002-omar-hassan-backdoor.md
│   ├── F003-mkfs-azfs-socat.md
│   └── ...                        ← One file per discrete finding
├── changes/
│   └── changelog.md               ← Single running log of every change made
├── credentials/
│   └── creds.md                   ← Password tracker (keep SECURE, offline)
├── runbooks/
│   ├── snapshot-procedure.md
│   ├── service-down-recovery.md
│   └── ...
└── scoring/
    └── service-uptime-checks.md
