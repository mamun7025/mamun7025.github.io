# VPS Server Troubleshoot & Security Notes

**Project:** VPS Server Auth and Spike  
**Purpose:** Security investigation, troubleshooting, and hardening checklist  
**VPS Hostname:** `vmi2697087`  
**Environment:** Nginx → Tomcat → Java/Grails application → MySQL  
**VPS RAM:** ~12 GB

> **Important:** During an incident, collect evidence before deleting users, removing SSH keys, rebooting, or changing configuration. Do not paste private SSH keys, passwords, tokens, or other secrets into chat.

---

## 1. Current Investigation Summary

The VPS has been receiving automated SSH scanning/brute-force traffic.

Previously observed high-volume failed SSH attempts included:

| Source IP | Failed attempts |
|---|---:|
| `14.225.23.94` | 800 |
| `87.248.131.176` | 484 |
| `45.156.87.209` | 450 |

Other suspicious/failed source IPs were also observed.

Successful SSH logins were previously identified from:

| Source IP | Authentication |
|---|---|
| `5.192.91.140` | Password |
| `176.204.255.214` | SSH key |
| `5.192.79.113` | SSH key |

### Important unresolved question

Determine whether the successful root password login from:

```text
5.192.91.140
```

was legitimate.

A successful login does **not automatically prove compromise**, but an unexpected root password login must be investigated carefully.

---

## 2. Fail2Ban Current Status

Fail2Ban was confirmed running:

```bash
sudo systemctl status fail2ban
```

Status previously showed:

```text
Active: active (running)
```

Check it with:

```bash
sudo fail2ban-client status
```

Then:

```bash
sudo fail2ban-client status sshd
```

If the jail is named `ssh` instead:

```bash
sudo fail2ban-client status ssh
```

Additional checks:

```bash
sudo fail2ban-client get sshd banned
```

```bash
sudo fail2ban-client get sshd currently-banned
```

---

# 3. Step-by-Step Troubleshooting

## Step 0 — Record Current Server State

```bash
hostname
date
uptime
who
```

```bash
uname -a
```

```bash
cat /etc/os-release
```

---

## Step 1 — Check Currently Logged-In Users

```bash
who
```

```bash
w
```

```bash
last -ai | head -30
```

Look for unknown users, unexpected root sessions, unknown remote IP addresses, and login times that do not match your activity.

---

## Step 2 — Check Successful and Failed SSH Logins

```bash
sudo journalctl -u ssh --since "7 days ago" | grep -Ei "Accepted|Failed|Invalid"
```

If the service is named `sshd`:

```bash
sudo journalctl -u sshd --since "7 days ago" | grep -Ei "Accepted|Failed|Invalid"
```

Ubuntu/Debian authentication log:

```bash
sudo grep -Ei "Accepted|Failed|Invalid user" /var/log/auth.log | tail -100
```

If `/var/log/auth.log` does not exist:

```bash
sudo journalctl --since "7 days ago" | grep -Ei "sshd.*(Accepted|Failed|Invalid)"
```

Pay special attention to:

```text
Accepted password for root
Accepted publickey for root
```

Record exact timestamp, username, source IP, and authentication method.

---

## Step 3 — Count Failed SSH Attempts by IP

```bash
sudo grep "Failed password" /var/log/auth.log \
| awk '{print $(NF-3)}' \
| sort \
| uniq -c \
| sort -nr \
| head -30
```

If the log is unavailable:

```bash
sudo journalctl --since "7 days ago" \
| grep "Failed password" \
| awk '{print $(NF-3)}' \
| sort \
| uniq -c \
| sort -nr \
| head -30
```

---

## Step 4 — Check Fail2Ban

```bash
sudo systemctl status fail2ban
```

```bash
sudo systemctl enable --now fail2ban
```

```bash
sudo fail2ban-client status
```

```bash
sudo fail2ban-client status sshd
```

```bash
sudo grep -R "^\[sshd\]" /etc/fail2ban/ -n
```

```bash
sudo fail2ban-client -d | head -100
```

---

## Step 5 — Check Firewall

```bash
sudo ufw status verbose
```

```bash
sudo firewall-cmd --state 2>/dev/null
```

```bash
sudo nft list ruleset
```

Do not change firewall rules during evidence collection unless necessary.

---

## Step 6 — Check Open/Listening Ports

```bash
sudo ss -lntup
```

```bash
sudo ss -lnt
```

Pay particular attention to:

```text
22    SSH
80    HTTP
443   HTTPS
8080  Tomcat
3306  MySQL
```

Desired general architecture:

```text
Internet
   |
   +---- 80/443 ----> Nginx
                         |
                         +----> Tomcat :8080 (private/local)
                                      |
                                      +----> MySQL :3306 (private/local)
```

---

## Step 7 — Check Running Processes

```bash
ps aux --sort=-%cpu | head -30
```

```bash
ps aux --sort=-%mem | head -30
```

```bash
sudo systemctl --type=service --state=running
```

Look for unfamiliar processes or services.

---

## Step 8 — Check SSH Configuration

```bash
sudo sshd -T | grep -Ei "permitrootlogin|passwordauthentication|pubkeyauthentication|permitempty|port"
```

```bash
sudo grep -Ei "^(PermitRootLogin|PasswordAuthentication|PubkeyAuthentication|Port|AllowUsers|AllowGroups)" /etc/ssh/sshd_config
```

Do not modify these settings until the investigation is complete.

---

## Step 9 — Check Root SSH Keys

```bash
sudo ls -la /root/.ssh/
```

```bash
sudo cat /root/.ssh/authorized_keys
```

```bash
sudo stat /root/.ssh/authorized_keys
```

Look for unknown public keys and unexpected modification dates.

**Never share private SSH keys.**

---

## Step 10 — Check All Users

```bash
cut -d: -f1,3,7 /etc/passwd
```

```bash
awk -F: '$7 ~ /(bash|sh|zsh)$/ {print $1, $3, $6, $7}' /etc/passwd
```

```bash
getent group sudo
```

```bash
getent group wheel
```

```bash
sudo ls -la /etc/sudoers.d/
```

```bash
sudo grep -R "^[^#]" /etc/sudoers /etc/sudoers.d/ 2>/dev/null
```

Look for unexpected privileged users.

---

## Step 11 — Check Cron Jobs

```bash
sudo crontab -l
```

```bash
sudo ls -la /etc/cron.d/
```

```bash
sudo ls -la /etc/cron.daily/
```

```bash
sudo ls -la /etc/cron.hourly/
```

```bash
sudo ls -la /etc/cron.weekly/
```

```bash
sudo ls -la /etc/cron.monthly/
```

```bash
sudo grep -R "" /etc/cron.d/ 2>/dev/null
```

Look for unfamiliar commands, scripts, downloaded files, or network commands.

---

## Step 12 — Check Systemd Services and Timers

```bash
systemctl list-unit-files --type=service --state=enabled
```

```bash
sudo find /etc/systemd/system /usr/lib/systemd/system \
-type f -printf '%TY-%Tm-%Td %TH:%TM %p\n' \
2>/dev/null | sort -r | head -50
```

```bash
sudo systemctl list-timers --all
```

---

## Step 13 — Check Recently Modified Files

```bash
sudo find /etc /root /var/www /opt \
-type f -mtime -7 \
-printf '%TY-%Tm-%Td %TH:%TM %p\n' \
2>/dev/null | sort -r | head -100
```

```bash
sudo find /root -type f -mtime -14 -ls 2>/dev/null
```

```bash
sudo find /etc/ssh -type f -mtime -30 -ls
```

Correlate modification times with suspicious SSH login times.

---

## Step 14 — Check Shell History

```bash
sudo cat /root/.bash_history
```

```bash
sudo find /home -maxdepth 2 -name ".bash_history" -type f -print
```

```bash
sudo cat /home/*/.bash_history 2>/dev/null
```

Shell history is not a complete forensic record. Missing history does not prove that no commands were executed.

---

## Step 15 — Check Sudo Activity

```bash
sudo journalctl --since "7 days ago" | grep -Ei "sudo|COMMAND="
```

```bash
sudo grep -Ei "sudo|COMMAND=" /var/log/auth.log | tail -100
```

Look for commands executed around suspicious login times.

---

## Step 16 — Check Nginx Access Logs

```bash
sudo tail -100 /var/log/nginx/access.log
```

```bash
sudo tail -100 /var/log/nginx/error.log
```

HTTP status-code distribution:

```bash
sudo awk '{print $9}' /var/log/nginx/access.log \
| sort | uniq -c | sort -nr
```

Top source IPs:

```bash
sudo awk '{print $1}' /var/log/nginx/access.log \
| sort | uniq -c | sort -nr | head -30
```

Top requested URLs:

```bash
sudo awk '{print $7}' /var/log/nginx/access.log \
| sort | uniq -c | sort -nr | head -30
```

---

## Step 17 — Search Nginx Logs for Common Attack Scans

```bash
sudo grep -Ei \
"wp-admin|wp-login|xmlrpc|\.env|phpmyadmin|cgi-bin|shell|cmd=|wget|curl|passwd|etc/passwd|\.git" \
/var/log/nginx/access.log \
| tail -100
```

These requests may be automated Internet scanning and do not by themselves prove compromise.

---

## Step 18 — Check Tomcat

```bash
sudo systemctl status tomcat
```

If the service has another name:

```bash
systemctl list-units --type=service | grep -i tomcat
```

```bash
sudo ss -lntp | grep 8080
```

```bash
sudo find /var/log -iname "*tomcat*" -type f 2>/dev/null
```

```bash
sudo find /opt /var/lib -iname "catalina.out" -type f 2>/dev/null
```

---

## Step 19 — Check MySQL Exposure

```bash
sudo ss -lntp | grep 3306
```

```bash
sudo grep -R "bind-address" /etc/mysql/ 2>/dev/null
```

MySQL should generally be bound to localhost/private interfaces unless remote database access is intentionally required.

---

## Step 20 — Check Current Network Connections

```bash
sudo ss -tunap
```

```bash
sudo lsof -i -n -P
```

Look for unknown processes, unexpected outbound connections, strange remote IPs, or unexpected listening services.

---

## Step 21 — Check Disk Space

```bash
df -h
```

```bash
df -ih
```

```bash
sudo du -xhd1 / | sort -h
```

---

## Step 22 — Check Recently Installed/Upgraded Packages

Ubuntu/Debian:

```bash
grep -Ei " install | upgrade " /var/log/dpkg.log | tail -100
```

```bash
grep -Ei " install | upgrade " /var/log/apt/history.log | tail -100
```

---

## Step 23 — Check Reboot History

```bash
last reboot
```

```bash
journalctl --list-boots
```

---

## Step 24 — Check System Errors

```bash
sudo journalctl -p warning..alert --since "7 days ago"
```

```bash
sudo dmesg --level=err,warn | tail -100
```

---

# 4. Security Investigation Correlation

The most important analysis is to correlate:

```text
Successful SSH login
        |
        v
Source IP
        |
        v
Exact timestamp
        |
        v
Commands executed
        |
        v
Files modified
        |
        v
SSH keys changed?
        |
        v
New users?
        |
        v
New cron jobs?
        |
        v
New systemd services?
        |
        v
Unexpected network connections?
```

Example:

```text
Accepted password for root from 5.192.91.140
        |
        +--> Check ±30 minutes around the login
        |
        +--> Check sudo/command activity
        |
        +--> Check file modification times
        |
        +--> Check authorized_keys
        |
        +--> Check cron/systemd
        |
        +--> Check processes/network connections
```

---

# 5. Hardening — Do After Evidence Collection

Do not make disruptive changes until the investigation is complete.

## 5.1 Disable root password SSH access

A common target is:

```text
PermitRootLogin prohibit-password
```

or, if root SSH access is not required:

```text
PermitRootLogin no
```

## 5.2 Disable SSH password authentication

After confirming that SSH key access works:

```text
PasswordAuthentication no
```

Keep:

```text
PubkeyAuthentication yes
```

**Always test a second SSH session before closing the current administrative session.**

## 5.3 Keep Fail2Ban enabled

```bash
sudo systemctl enable --now fail2ban
```

```bash
sudo fail2ban-client status
```

## 5.4 Restrict Firewall

Expose only services that are genuinely required.

Typical public services:

```text
22   SSH
80   HTTP
443  HTTPS
```

Backend services such as:

```text
8080 Tomcat
3306 MySQL
```

should normally remain private/local.

---

# 6. Recommended Application Architecture

```text
                    INTERNET
                       |
                 +-----+-----+
                 |           |
               HTTP        HTTPS
                 |           |
                 +-----+-----+
                       |
                    NGINX
                    :80/:443
                       |
                       v
                 TOMCAT / APP
                    :8080
                 (private/local)
                       |
                       v
                    MYSQL
                    :3306
                 (private/local)
```

SSH administration:

```text
Administrator
      |
      v
    SSH :22
      |
      v
    VPS
```

SSH should be protected with SSH keys, reduced root access, Fail2Ban, firewall rules, and strong account security.

---

# 7. Safe Investigation Rules

### Do

- Record timestamps.
- Save relevant logs before changing configuration.
- Compare IPs with your known access.
- Check SSH keys.
- Check users and sudo privileges.
- Check cron and systemd.
- Check recently modified files.
- Check outbound network connections.
- Check Nginx/Tomcat logs.
- Keep Fail2Ban enabled.

### Don't

- Do not delete suspicious files immediately.
- Do not remove unknown users immediately.
- Do not remove SSH keys before recording them.
- Do not reboot before collecting evidence unless necessary.
- Do not disable SSH access while connected without a tested alternative.
- Do not paste private keys/passwords/tokens into chat.

---

# 8. First Commands to Run

If starting the investigation again, begin with these:

```bash
who
```

```bash
w
```

```bash
last -ai | head -50
```

```bash
sudo journalctl -u ssh --since "7 days ago" | grep -Ei "Accepted|Failed|Invalid"
```

```bash
sudo fail2ban-client status
```

```bash
sudo fail2ban-client status sshd
```

Then continue with the remaining sections.

---

# 9. Investigation Status Template

## Server

- Hostname:
- OS:
- Kernel:
- Uptime:
- Public IP:

## SSH

- Root password login enabled:
- Password authentication enabled:
- SSH key authentication enabled:
- Root authorized keys reviewed:
- Unknown users:
- Unknown SSH keys:

## Firewall

- UFW:
- Firewalld:
- Nftables:
- Public ports:

## Fail2Ban

- Service running:
- SSH jail:
- Currently banned:
- Total banned:

## Services

- Nginx:
- Tomcat:
- MySQL:
- Other services:

## Suspicious Activity

- Failed SSH attempts:
- Successful SSH logins:
- Unknown source IPs:
- Unknown processes:
- Unknown outbound connections:
- Recently modified suspicious files:
- Unknown cron jobs:
- Unknown systemd services:

## Final Assessment

- [ ] No suspicious successful login identified
- [ ] All successful logins verified
- [ ] SSH keys verified
- [ ] Users verified
- [ ] Sudo configuration verified
- [ ] Cron verified
- [ ] Systemd verified
- [ ] Network listeners verified
- [ ] Nginx logs reviewed
- [ ] Tomcat logs reviewed
- [ ] MySQL exposure verified
- [ ] Fail2Ban verified
- [ ] Firewall verified
- [ ] SSH hardened
