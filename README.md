# Task 4: Setup and Use a Firewall on Linux (UFW on Kali)

## Objective
Configure and test basic firewall rules to allow or block network traffic using UFW (Uncomplicated Firewall) on Kali Linux.

## Environment
- Kali Linux VM (IP: 192.168.29.217 on eth0)
- Windows host (used as the remote test machine)
- Tools: UFW, netcat (nc), nmap, PowerShell Test-NetConnection

## Steps Performed

### 1. Install UFW and list current rules
```bash
sudo apt update && sudo apt install ufw -y
sudo ufw status verbose | tee 01_ufw_status_before.txt
```
UFW was inactive initially.

### 2. Set default policies
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 3. Allow SSH (port 22) before enabling
```bash
sudo ufw allow 22/tcp
```

### 4. Enable firewall and list rules
```bash
sudo ufw enable
sudo ufw status numbered | tee 02_ufw_status_enabled_ssh_allowed.txt
```

### 5. Start a test listener on port 23 (Kali terminal 2)
```bash
sudo nc -lvnp 23
```

### 6. Test before blocking
```bash
nmap -p 23 192.168.29.217 | tee 03_nmap_before_block.txt
```

### 7. Add rule to block inbound Telnet (port 23)
```bash
sudo ufw deny 23/tcp
sudo ufw status numbered | tee 04_ufw_status_with_block.txt
```

### 8. Test the block
From Kali:
```bash
nmap -p 23 192.168.29.217 | tee 05_nmap_after_block.txt
```
Result: 23/tcp filtered telnet.

From the Windows host (PowerShell):
```powershell
Test-NetConnection -ComputerName 192.168.29.217 -Port 23 | Tee-Object -FilePath 06_windows_test_blocked.txt
```
Result: TcpTestSucceeded : False.

Note: UFW does not filter loopback (127.0.0.1), so tests must use the real IP or another machine.

### 9. Check the firewall log
Kali has no /var/log/ufw.log by default, so the kernel log is used:
```bash
sudo ufw logging on
sudo journalctl -k | grep "UFW BLOCK" | tail -n 20 | tee 07_ufw_block_log.txt
```
Entries show [UFW BLOCK] ... DPT=23.

### 10. Remove the test block rule
```bash
sudo ufw delete deny 23/tcp
sudo ufw status verbose | tee 08_ufw_status_final.txt
nmap -p 23 192.168.29.217 | tee 09_nmap_after_rule_removed.txt
```
Port 23 is reachable again, confirming the rule caused the block.

### 11. Export configuration
```bash
sudo cp /etc/ufw/user.rules ./user.rules
sudo cp /etc/ufw/user6.rules ./user6.rules
```

## Repository Contents
| File | Description |
|------|-------------|
| README.md | This document |
| commands.sh | All commands used, in order |
| 00_ip_address.txt | Kali network info |
| 01_ufw_status_before.txt | Rules before changes |
| 02_ufw_status_enabled_ssh_allowed.txt | After enabling UFW and allowing SSH |
| 03_nmap_before_block.txt | Port 23 scan before block |
| 04_ufw_status_with_block.txt | Rules with port 23 denied |
| 05_nmap_after_block.txt | Port 23 scan after block (filtered) |
| 06_windows_test_blocked.txt | Windows remote test (blocked) |
| 07_ufw_block_log.txt | UFW BLOCK log entries |
| 08_ufw_status_final.txt | Final state after removing the test rule |
| 09_nmap_after_rule_removed.txt | Scan after rule removal |
| user.rules, user6.rules | UFW configuration files |

## How a Firewall Filters Traffic
A firewall inspects each packet and compares it against an ordered rule list (source/destination IP, port, protocol, direction). The first matching rule decides the action: allow, deny (silently drop) or reject (drop and notify). If no rule matches, the default policy applies (here: deny incoming, allow outgoing). UFW is a front-end for the kernel's Netfilter (iptables/nftables), which does the actual filtering. Blocking port 23 stops unencrypted Telnet, while allowing port 22 keeps encrypted SSH available.

## Outcome
Learned to install, configure, test and remove UFW rules, and how packet filtering controls network traffic.
