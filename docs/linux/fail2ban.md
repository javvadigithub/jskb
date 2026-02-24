# Fail2Ban – Intrusion Prevention Framework for Linux

---

## 1. Concept

Fail2Ban is a log-monitoring intrusion prevention system that protects Linux servers against brute-force attacks.

It works by:

* Monitoring log files (e.g., `/var/log/auth.log`)
* Detecting repeated failed login attempts
* Automatically banning offending IP addresses
* Updating firewall rules dynamically

Primary use case: Protecting SSH from brute-force attacks.

---

## 2. Architecture / Working Principle

```
Attacker → SSH Service → Failed Log Entries
                           ↓
                    Fail2Ban Monitor
                           ↓
                  Regex Pattern Match
                           ↓
                  Firewall Rule Insert
                           ↓
                     IP Address Banned
```

---

### Core Components

| Component | Purpose                         |
| --------- | ------------------------------- |
| Jail      | Defines service to monitor      |
| Filter    | Regex pattern to detect attacks |
| Log Path  | Log file being monitored        |
| Action    | Firewall action (ban IP)        |

---

## 3. Installation

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install fail2ban -y
```

### RHEL / CentOS

```bash
sudo yum install epel-release -y
sudo yum install fail2ban -y
```

---

## 4. Configuration

Never modify `jail.conf` directly.

Instead create:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

---

### Basic SSH Protection Configuration

Edit:

```bash
sudo vi /etc/fail2ban/jail.local
```

Add or modify:

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 600
findtime = 600
```

---

### Parameter Meaning

| Parameter | Description                  |
| --------- | ---------------------------- |
| maxretry  | Failed attempts before ban   |
| bantime   | Ban duration (seconds)       |
| findtime  | Time window to count retries |

---

## 5. Start and Enable Service

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Check status:

```bash
sudo systemctl status fail2ban
```

---

## 6. Check Active Jails

```bash
sudo fail2ban-client status
```

Check SSH jail specifically:

```bash
sudo fail2ban-client status sshd
```

---

## 7. Unban an IP Address

```bash
sudo fail2ban-client set sshd unbanip 192.168.1.10
```

---

## 8. Security Implications

### Defensive Perspective

* Protects SSH from brute-force attacks
* Reduces automated scanning success
* Adds automated incident response capability

---

### Red Team Perspective

* Can detect brute-force attempts quickly
* Requires distributed attack or slow brute-force to bypass
* IP rotation can evade detection

---

## 9. Integration with Firewall

Fail2Ban integrates with:

* iptables
* nftables
* firewalld
* UFW

You can check current bans:

```bash
sudo iptables -L
```

---

## 10. Log Location

Main log file:

```
/var/log/fail2ban.log
```

View logs:

```bash
sudo tail -f /var/log/fail2ban.log
```

---

## 11. Lab Demonstration

1. Enable SSH jail
2. Attempt 5 wrong SSH logins
3. Check status:

```bash
sudo fail2ban-client status sshd
```

You should see your IP listed under banned IP list.

---

## 12. Risk & Misconfiguration

| Risk                | Impact                   |
| ------------------- | ------------------------ |
| Too strict maxretry | Legitimate users blocked |
| Long bantime        | Lockout risk             |
| Wrong logpath       | Jail ineffective         |

---

## 13. Summary

* Fail2Ban is reactive intrusion prevention.
* It monitors logs, not traffic directly.
* It dynamically updates firewall rules.
* Commonly used to protect SSH.
* Not a replacement for proper hardening.

---

# 🔧 Fail2Ban Configuration File Structure

Fail2Ban uses a layered configuration system.

You should **never edit core `.conf` files directly**.

---

## 1️⃣ Main Configuration Directory

```
/etc/fail2ban/
```

Inside this directory:

```
/etc/fail2ban/
├── fail2ban.conf
├── jail.conf
├── jail.local
├── filter.d/
├── action.d/
└── jail.d/
```

---

# 2️⃣ Core Configuration Files Explained

---

## 🔹 1. `/etc/fail2ban/fail2ban.conf`

**Purpose:**
Main daemon configuration.

Controls:

* Logging
* Socket file
* Log level
* PID file
* Database backend

Example:

```ini
[Definition]
loglevel = INFO
logtarget = /var/log/fail2ban.log
socket = /var/run/fail2ban/fail2ban.sock
pidfile = /var/run/fail2ban/fail2ban.pid
```

⚠ Normally you do NOT modify this.

---

## 🔹 2. `/etc/fail2ban/jail.conf`

**Purpose:**
Default jail configuration for all services.

Contains:

* Default settings
* Predefined jails (sshd, apache, vsftpd, etc.)

Example:

```ini
[DEFAULT]
bantime  = 600
findtime = 600
maxretry = 5

[sshd]
enabled  = false
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
```

⚠ Never edit this file directly. It will be overwritten during updates.

---

## 🔹 3. `/etc/fail2ban/jail.local`  ✅ (Recommended)

This file overrides `jail.conf`.

Create it:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Or better:

```bash
sudo nano /etc/fail2ban/jail.local
```

Example minimal config:

```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
ignoreip = 127.0.0.1/8 192.168.1.0/24

[sshd]
enabled = true
```

---

## 🔹 4. `/etc/fail2ban/jail.d/`

Used for modular jail configurations.

Instead of putting everything in `jail.local`, you can create:

```
/etc/fail2ban/jail.d/sshd.local
```

Example:

```ini
[sshd]
enabled = true
maxretry = 3
bantime = 7200
```

Professional setups prefer this modular method.

---

## 🔹 5. `/etc/fail2ban/filter.d/`

Contains regex filters used to detect malicious activity.

Example:

```
/etc/fail2ban/filter.d/sshd.conf
```

Inside:

```ini
[Definition]
failregex = ^%(__prefix_line)sFailed password for .* from <HOST>
ignoreregex =
```

You can create custom filters here.

Example:

```
/etc/fail2ban/filter.d/custom-ssh.conf
```

---

## 🔹 6. `/etc/fail2ban/action.d/`

Defines what action happens when a ban is triggered.

Examples:

* iptables-multiport.conf
* ufw.conf
* firewallcmd.conf

Example:

```ini
[Definition]
actionstart = iptables -N fail2ban-<name>
actionban   = iptables -I fail2ban-<name> 1 -s <ip> -j DROP
```

---

# 3️⃣ Configuration Hierarchy (Important)

Fail2Ban loads config in this order:

```
1. *.conf (default files)
2. *.local (overrides)
3. jail.d/*.conf
4. jail.d/*.local
```

Priority rule:

```
.local overrides .conf
```

---

# 4️⃣ Advanced: Recidive Jail

Recidive tracks repeated offenders across jails.

Enable in `jail.local`:

```ini
[recidive]
enabled  = true
logpath  = /var/log/fail2ban.log
banaction = iptables-allports
findtime = 86400
bantime  = 604800
maxretry = 5
```

What this does:

* Monitors Fail2Ban log
* If IP gets banned multiple times
* Permanent or long-term ban applied

---

# 5️⃣ Recommended Production Setup

Instead of editing `jail.local`, do:

```
/etc/fail2ban/jail.d/
```

Create separate files:

```
sshd.local
recidive.local
apache.local
```

Keeps configuration modular and audit-friendly.

---

# 6️⃣ Configuration Validation

Check syntax:

```bash
sudo fail2ban-client -d
```

Reload config safely:

```bash
sudo systemctl reload fail2ban
```

---

# 7️⃣ Security Perspective

Misconfiguration risks:

| Risk               | Impact                    |
| ------------------ | ------------------------- |
| Editing jail.conf  | Overwritten during update |
| No ignoreip set    | Self-lockout risk         |
| Incorrect logpath  | Jail never triggers       |
| Wrong filter regex | False negatives           |

---

## Visual Overview

[![Shred Concept Diagram](../../assets/images/fail2ban.png)](../../assets/images/fail2ban.png)

