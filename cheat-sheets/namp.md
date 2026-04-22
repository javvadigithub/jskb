````markdown id="f5vz1k"
# Nmap Cheat Sheet

## Basic Syntax

```bash
nmap [scan type] [options] <target>
````

Examples:

```bash
nmap 192.168.1.1
nmap scanme.nmap.org
nmap 192.168.1.0/24
```

---

# Target Specification

## Single IP

```bash
nmap 10.10.10.5
```

## Multiple IPs

```bash
nmap 10.10.10.5 10.10.10.10
```

## Range

```bash
nmap 10.10.10.1-50
```

## Subnet

```bash
nmap 10.10.10.0/24
```

## File Input

```bash
nmap -iL targets.txt
```

---

# Host Discovery

## Ping Scan (Discover Live Hosts Only)

```bash
nmap -sn 192.168.1.0/24
```

## Disable Ping (Treat Host as Up)

```bash
nmap -Pn 10.10.10.5
```

## ARP Scan (Local LAN)

```bash
nmap -PR 192.168.1.0/24
```

---

# Port Scanning

## Default Scan (Top 1000 Ports)

```bash
nmap 10.10.10.5
```

## Scan Specific Port

```bash
nmap -p 80 10.10.10.5
```

## Scan Multiple Ports

```bash
nmap -p 22,80,443 10.10.10.5
```

## Scan Port Range

```bash
nmap -p 1-1000 10.10.10.5
```

## Scan All Ports

```bash
nmap -p- 10.10.10.5
```

## Fast Scan

```bash
nmap -F 10.10.10.5
```

---

# Scan Types

## TCP SYN Scan (Stealth)

```bash
sudo nmap -sS 10.10.10.5
```

## TCP Connect Scan

```bash
nmap -sT 10.10.10.5
```

## UDP Scan

```bash
sudo nmap -sU 10.10.10.5
```

## ACK Scan

```bash
sudo nmap -sA 10.10.10.5
```

## FIN Scan

```bash
sudo nmap -sF 10.10.10.5
```

## NULL Scan

```bash
sudo nmap -sN 10.10.10.5
```

## Xmas Scan

```bash
sudo nmap -sX 10.10.10.5
```

---

# Service & Version Detection

## Detect Services

```bash
nmap -sV 10.10.10.5
```

## Aggressive Version Detection

```bash
nmap -sV --version-intensity 9 10.10.10.5
```

---

# OS Detection

```bash
sudo nmap -O 10.10.10.5
```

## Aggressive Scan

```bash
sudo nmap -A 10.10.10.5
```

Includes:

* OS Detection
* Version Detection
* Script Scan
* Traceroute

---

# NSE (Nmap Scripting Engine)

## Default Scripts

```bash
nmap -sC 10.10.10.5
```

## Run Specific Script

```bash
nmap --script http-title 10.10.10.5
```

## Vulnerability Scan

```bash
nmap --script vuln 10.10.10.5
```

## SMB Enumeration

```bash
nmap --script smb-enum-shares -p445 10.10.10.5
```

## FTP Anonymous Login Check

```bash
nmap --script ftp-anon -p21 10.10.10.5
```

## SSH Algorithms

```bash
nmap --script ssh2-enum-algos -p22 10.10.10.5
```

---

# Timing & Speed

## Faster Scan

```bash
nmap -T4 10.10.10.5
```

## Fastest

```bash
nmap -T5 10.10.10.5
```

## Slow / Stealthier

```bash
nmap -T1 10.10.10.5
```

Timing Levels:

* T0 Paranoid
* T1 Sneaky
* T2 Polite
* T3 Normal
* T4 Aggressive
* T5 Insane

---

# Output Options

## Normal Output

```bash
nmap -oN result.txt 10.10.10.5
```

## XML Output

```bash
nmap -oX result.xml 10.10.10.5
```

## Grepable Output

```bash
nmap -oG result.grep 10.10.10.5
```

## All Formats

```bash
nmap -oA scanresult 10.10.10.5
```

Creates:

* scanresult.nmap
* scanresult.xml
* scanresult.gnmap

---

# Firewall Evasion / Stealth

## Fragment Packets

```bash
sudo nmap -f 10.10.10.5
```

## Spoof Source IP

```bash
sudo nmap -S 1.2.3.4 10.10.10.5
```

## Decoy Scan

```bash
sudo nmap -D RND:10 10.10.10.5
```

## Randomize Hosts

```bash
nmap --randomize-hosts 10.10.10.0/24
```

---

# Useful Real World Commands

## Find Open Web Ports

```bash
nmap -p80,443,8080,8443 10.10.10.5
```

## Full TCP + Version Scan

```bash
sudo nmap -sS -sV -p- 10.10.10.5
```

## Quick Internal Network Discovery

```bash
nmap -sn 192.168.1.0/24
```

## Detect SMB Servers

```bash
nmap -p445 --open 192.168.1.0/24
```

## Detect RDP Systems

```bash
nmap -p3389 --open 192.168.1.0/24
```

---

# Common Ports to Remember

| Port | Service |
| ---- | ------- |
| 21   | FTP     |
| 22   | SSH     |
| 23   | Telnet  |
| 25   | SMTP    |
| 53   | DNS     |
| 80   | HTTP    |
| 110  | POP3    |
| 139  | NetBIOS |
| 143  | IMAP    |
| 443  | HTTPS   |
| 445  | SMB     |
| 3306 | MySQL   |
| 3389 | RDP     |

---

# CEH Exam Tips

## SYN Scan

```bash
-sS
```

Most common stealth scan.

## UDP Scan

```bash
-sU
```

Used for DNS / SNMP / TFTP.

## Service Detection

```bash
-sV
```

## OS Detection

```bash
-O
```

## Aggressive Scan

```bash
-A
```

---

# Quick Combo Commands

## Safe Enumeration

```bash
nmap -sC -sV target
```

## Deep Scan

```bash
sudo nmap -A -p- target
```

## Vulnerability Scripts

```bash
nmap --script vuln target
```

---

# Final Reminder

Use Nmap only on systems you own or have permission to assess.

```
```
## Visual Overview

[![Shred Concept Diagram](../../assets/images/nmap_scan.png)](../../assets/images/nmap_scan.png)