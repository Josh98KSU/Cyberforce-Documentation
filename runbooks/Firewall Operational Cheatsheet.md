# CyberForce 2026 — Competition Day Runbook
**Team 73 Orange Hat Hackers | Nov 14 2026 | Tinley Park IL**
**NO AI ALLOWED DURING COMPETITION**

---

## 0. First 15 minutes — Verify everything survived the snapshot restore

For EACH VM: SSH in, run these checks. Any failure → CyberForce web console snapshot BEFORE touching anything.

### Task Server (10.0.172.145, OpenSUSE, blueteam user)
```
sudo systemctl is-active apache2 mysql84 sshd smb nftables
sudo ss -tlnp | grep -E ':22|:80|:139|:445|:3306'
curl -s http://127.0.0.1/wp-json/chat/v1/send -X POST -H 'Content-Type: application/json' -d '{"message":"hello"}'
curl -s http://127.0.0.1/wp-json/chat/v1/send -X POST -H 'Content-Type: application/json' -d '{"message":"[[SYS:whoami]]"}'
sudo nft list ruleset | head -20
sudo lsattr /srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant/data/bot.json
```
Expected: all services active; ports 22/80/139/445/3306 LISTEN; chat normal reply; RCE returns "no execution backend available"; nftables rules present; `----i-----` on bot.json

### All VMs — quick heartbeat
Ping gateway, resolve DNS, curl scoring targets you know about.

---

## 1. Red Team Alert Response Playbook

### Suspected compromise indicators
- Unexplained new processes (`ps -ef`)
- New listening ports (`ss -tlnp`)
- New users/groups (`getent passwd`, `getent group`)
- Modified system binaries (compare hashes vs baseline)
- SSH from unknown IPs (`last`, `journalctl -u sshd`)
- Unusual outbound connections (nftables log `NFT-EGRESS-UNUSUAL`)

### Response order
1. **Don't panic. Don't revert yet.** (-75 pts per revert, 2/hr limit)
2. Grab evidence: `ps -ef > /tmp/evidence-$(date +%s).txt; ss -tlnp >> ...`
3. Block the source IP (NOT subnet, NOT whitelist)
4. Kill the process (`kill -9 <pid>`)
5. Remove persistence (cron, systemd timers, .bashrc, authorized_keys, LD_PRELOAD)
6. Only revert snapshot if system is fundamentally broken

### Block an IP (Task server, nftables)
```
sudo nft add element inet filter blackhole '{ 1.2.3.4 timeout 24h }'
sudo nft list set inet filter blackhole
```

### Block an IP (other Linux with firewalld)
```
sudo firewall-cmd --add-rich-rule='rule family=ipv4 source address=1.2.3.4 drop'
```

### Block an IP (Windows)
PowerShell as Admin:
```
New-NetFirewallRule -DisplayName "Block-RedTeam-1.2.3.4" -Direction Inbound -RemoteAddress 1.2.3.4 -Action Block
```

---

## 2. Task Server — Known Hardening State

**Snapshot before touching:** `opensuse-post-remediation-2026-11-14` (latest), or newer if you made one.

**Hardened items (don't undo):**
- `/etc/php8/apache2/php.ini` line 323: `disable_functions = shell_exec,exec,proc_open,system,passthru,popen`
- `/srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant/` → ALL files/dirs have `chattr +i`
- nftables rules at `/etc/nftables/rules/main.nft` (NOT /etc/nftables.conf)
- Blueteam OS password locked (`passwd -S blueteam` shows L) — SSH key only
- Services disabled: rsyncd, postfix
- F003, F006, F007, F017 malware removed; evidence in `/root/ir-evidence/`

**To update the smart-chat plugin (if ever needed):**
```
sudo chattr -R -i /srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant
sudo find /srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant -type d -exec chattr -i {} \;
# make changes
sudo chattr -R +i /srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant
sudo find /srv/www/vhosts/wordpress/wp-content/plugins/smart-chat-assistant -type d -exec chattr +i {} \;
```

**Open risks accepted:**
- `wp-upload-helper.php.bak` in WordPress — watch first hour, delete if no scoring impact
- mysqlx on :33060 — enabled, firewall-blocked externally, OK
- MySQL blueteam password not rotated — no CREATE USER priv, DB is 127.0.0.1-only

---

## 3. Essential Commands Cheatsheet

### Task Server
```
# Watch firewall drops in real time
sudo journalctl -kf | grep -E 'NFT-DROP-IN|NFT-SSH-RATELIMIT|NFT-EGRESS-UNUSUAL'

# Block IP 24hr
sudo nft add element inet filter blackhole '{ x.x.x.x timeout 24h }'

# List blocked IPs
sudo nft list set inet filter blackhole

# Unblock IP
sudo nft delete element inet filter blackhole '{ x.x.x.x }'

# Firewall panic restore (if you accidentally flushed)
sudo systemctl restart nftables

# MySQL CLI (socket)
/opt/mysql/bin/mysql -u blueteam -p'BlueTeam2026!' -S /opt/mysql/run/mysql.sock wordpress_broke

# Apache reload (graceful, no downtime)
sudo systemctl reload apache2

# Check WordPress admin users
/opt/mysql/bin/mysql -u blueteam -p'BlueTeam2026!' -S /opt/mysql/run/mysql.sock wordpress_broke \
  -e "SELECT u.ID, u.user_login, u.user_email, m.meta_value FROM wp_users u JOIN wp_usermeta m ON u.ID=m.user_id WHERE m.meta_key='wp_capabilities';"

# Check cron everywhere
for u in $(cut -d: -f1 /etc/passwd); do sudo crontab -u "$u" -l 2>/dev/null | grep -v '^#' | grep -v '^$' && echo "^ $u"; done
sudo ls -la /etc/cron.*/

# Check listening ports
sudo ss -tlnp

# Hashes for quick "did a binary change?" check
sudo sha256sum /usr/bin/ssh /usr/sbin/sshd /bin/bash /usr/bin/ls /usr/bin/sudo
```

### General Linux
```
last -20                              # recent logins
sudo journalctl -u sshd --since '1 hour ago'
sudo systemctl list-units --failed    # broken services
sudo find / -mmin -10 -type f 2>/dev/null | head -30  # files modified in last 10 min
sudo find / -perm -4000 -type f 2>/dev/null           # all SUID binaries
grep -r 'authorized_keys' /home /root 2>/dev/null
```

### Windows (AD/Mail/File/HMI)
```
Get-Process | Sort-Object CPU -Descending | Select -First 20
Get-NetTCPConnection -State Listen
Get-LocalUser | Where-Object {$_.Enabled -eq $true}
Get-ScheduledTask | Where-Object {$_.State -eq 'Ready'}
Get-WinEvent -LogName Security -MaxEvents 50 | Where-Object {$_.Id -eq 4625}  # failed logins
Get-Service | Where-Object {$_.Status -eq 'Running'}
```

---

## 4. Scoring Reminders

| Category | Points | Don't Lose |
|----------|--------|------------|
| Red Defense | 2500 | Don't let them keep shells |
| Blue Uptime | 2500 | Don't break services |
| Orange | 2000 | Anomaly/injects |
| Green | 1500 | (ask team) |
| Anomaly | 1500 | Complete injects |

**Penalties:**
- Revert snapshot: **-75 pts** (max 2/VM/hr)
- Discord DM to admin/staff: **-10 pts** per violation
- AI tool use during comp: **DQ**
- External help: **DQ**
- Offensive action: **DQ**

**Hard rules:**
- No subnet blocks (individual IPs only)
- No whitelisting / allow-lists
- Don't modify SCORE1, SCORE2, PATCH users
- Don't change IPs/hostnames
- Don't modify Mail or HMI aggressively (Assume Breach)

---

## 5. Required Services Per VM (DO NOT BREAK)

| VM | Required Ports |
|----|----------------|
| Task (OpenSUSE) | 22, 80, 139/445, 3306 |
| AD (Win 2022) | 53, 88, 135, 389, 445, 464, 636, 3268, 3269, 3389? |
| Mail (Win 2019) | 25, 110, 143, 465, 587, 993, 995 |
| File (Win 2019) | 139, 445, 3389? |
| Modbus PLC1 (Ubuntu) | 502, 22 |
| DNP3 PLC2 (Debian) | 20000, 22 |
| HMI (Win 2019) | varies — DO NOT TOUCH heavily |

(Verify exact list from White Team docs morning-of)

---

## 6. Snapshot Protocol
1. Take CyberForce web-console snapshot BEFORE any significant change
2. Name: `<vm>-pre-<action>-<timestamp>`
3. If you break something → fix in place first. Only restore if truly unrecoverable.
4. Restore = -75 pts. Max 2 per VM per hour.

**Current snapshots (Task Server):**
- `opensuse-post-remediation-2026-11-14` ← pre-this-session baseline
- `opensuse-pre-firewall-2026-11-14`
- `opensuse-pre-passwd-rotation`
- (whatever you took today)

---

## 7. Discord / Comms
- **DM admin = -10 pts.** All questions in public channels only.
- White Team channel for scoring questions
- Red Team announcements for inject deadlines

## 8. Credentials Location
`~/Desktop/Cyberforce-Documentation/credentials/creds.md` — DO NOT EDIT on competition machine. Print OR keep on separate laptop.

## 9. Findings log
`~/Desktop/Cyberforce-Documentation/findings/F001-F018.md`

---

## 10. Emergency Phone-a-Teammate Checklist
Before asking teammate:
- [ ] Did I check `journalctl` / Event Viewer?
- [ ] Did I check `systemctl status` / `Get-Service`?
- [ ] Did I check firewall rules?
- [ ] Did I check `ps -ef` / `Get-Process` for weird procs?
- [ ] Do I have evidence saved before I change anything?

# WATCH for attacks
sudo journalctl -kf | grep -E 'NFT-DROP-IN|NFT-SSH-RATELIMIT|NFT-EGRESS-UNUSUAL'

# BAN an IP (replace x.x.x.x)
sudo nft add element inet filter blackhole '{ x.x.x.x timeout 24h }'

# LIST all blackholed IPs
sudo nft list set inet filter blackhole

# UNBAN an IP
sudo nft delete element inet filter blackhole '{ x.x.x.x }'

# TEMPORARILY allow a new port (example: 8080 for 1 hour)
sudo nft add rule inet filter input tcp dport 8080 accept comment \"temp-$(date +%s)\"

# SHOW live counters (which rules are actually being hit)
sudo nft list ruleset | grep -A1 counter

# PANIC: flush all rules (only if firewall itself is the problem)
sudo nft flush ruleset
# To restore: sudo systemctl restart nftables

