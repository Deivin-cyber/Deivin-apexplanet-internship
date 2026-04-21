
# Firewall Basics – Metasploitable2 Lab

## Author
Aman Jiwani  

## Lab Setup
- **Attacker VM:** Kali Linux  
- **Target VM:** Metasploitable2  
- **Network:** Host-only  

## Tool
- **Iptables**  
- **Date:** 24 Sep 2025  
- **Target IP:** 192.168.56.104  
- **Attacker IP:** 192.168.56.105  

---

## Lab Objective
- Create basic firewall rules.  
- Block specific attacker IP.  
- Filter unwanted traffic while allowing essential services.  

---

## Firewall Rules Applied

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `sudo iptables -F / -X` | Clear old rules |
| 2 | `sudo iptables -A INPUT -i lo -j ACCEPT` | Allow loopback |
| 3 | `sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT` | Allow established connections |
| 4 | `sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT` | Allow SSH |
| 5 | `sudo iptables -A INPUT -s 192.168.56.101 -j DROP` | Block attacker IP |
| 6 | `sudo iptables -A INPUT -j DROP` | Drop all other traffic |
| 7 | `sudo iptables -L -v -n --line-numbers` | Verify rules |

---

## Verification with Nmap

| Stage | Command | Screenshot |
|-------|---------|------------|
| Before firewall | `sudo nmap -Pn -sS -p- -T4 <TARGET_IP>` | `screenshots/nmap_before_block.png` |
| After firewall | `sudo nmap -Pn -sS -p- -T4 <TARGET_IP>` | `screenshots/nmap_after_block.png` |

> Many ports should show as timed out/filtered after firewall rules.

---

## Reset Firewall (if needed)
```bash
sudo iptables -F
sudo iptables -X
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
