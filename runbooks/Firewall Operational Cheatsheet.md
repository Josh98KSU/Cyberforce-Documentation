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
