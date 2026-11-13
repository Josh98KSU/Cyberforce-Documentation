# F002 — omar.hassan SSH Backdoor Account

**Discovered:** 2026-11-13 ~20:50 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** INVESTIGATE

## What It Is
User `omar.hassan` (UID 1085) has an SSH authorized_keys entry with the comment `red_team@kali`.

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGDbq+DD6puDMNC3l4idJ6VIlsVq8GtgJsaOxFX5JcdM red_team@kali
```

## Why Malicious
- Key comment literally says "red_team@kali"
- Account blends in with the employee naming pattern (many `firstname.lastname` users exist)
- Empty `.bash_history` = never been used or was cleared

## Open Question
Is `omar.hassan` listed as a legitimate employee in the C-Suite scenario document? If yes, the account is legit but compromised (remove key, lock account). If no, delete the account entirely.

## Remediation Plan
### If NOT a legitimate scenario user:
1. Lock account: `sudo passwd -l omar.hassan`
2. Kill any active sessions: `sudo pkill -KILL -u omar.hassan`
3. Archive home dir for evidence: `sudo tar czf /root/ir-evidence-omar.tar.gz /home/omar.hassan`
4. Delete user: `sudo userdel -r omar.hassan`

### If IS a legitimate scenario user:
1. Remove the SSH key: `sudo rm /home/omar.hassan/.ssh/authorized_keys`
2. Force password reset: `sudo passwd -e omar.hassan`
3. Monitor for re-addition

Note: Also check /etc/ssh/.keys/omar.hassan (see F009 — SSH redirect)

## Remediation Actions Taken
- [ ] Not yet started
