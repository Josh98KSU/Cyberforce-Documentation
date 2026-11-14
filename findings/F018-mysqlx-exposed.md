# F018 - MySQL X Protocol (mysqlx) enabled on port 33060
Status: RISK-ACCEPTED (firewall mitigation)
VM: Task (10.0.172.145)
Discovered: 2026-11-14 during service hardening pass
Severity: Low (firewall-contained)

## Observation
mysqlx plugin active in mysqld (MySQL 8.4), binds *:33060.

## Why not fully fixed
- blueteam MySQL user lacks DELETE on mysql.plugin (cannot UNINSTALL PLUGIN)
- mysqld restart to apply mysqlx=0 in my.cnf would cause ~2-5s WordPress DB downtime
- Risk/reward unfavorable given firewall already blocks external access

## Mitigation
nftables input chain policy=drop, explicit allow only 22/80/139/445/3306.
Verified externally from 10.0.172.175: port 33060 BLOCKED, port 3306 REACHABLE.

## Future remediation (if time permits)
1. Edit /opt/mysql/my.cnf, add `mysqlx=0` under [mysqld]
2. sudo systemctl restart mysql84
3. Verify: ss -tlnp | grep 33060  (should be empty)
