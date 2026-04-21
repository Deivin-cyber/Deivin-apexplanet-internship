# Nmap Port Scanning Lab Notes

## Author
Aman Jiwani

## Lab Setup
- **Attacker VM:** Kali Linux  
- **Target VM:** Metasploitable2 / Any Lab VM  
- **Network:** Host-only / Lab Network  

## Scan Tool
- **Tool:** Nmap  
- **Scan Date:** Tue, 22 Sep 2025  
- **Target IP:** 192.168.56.104  

---

## Scan Summary

| Scan Type | Purpose | Command | Explanation |
|-----------|---------|---------|-------------|
| Full TCP Port Scan | Discover all open TCP ports | `sudo nmap -Pn -p- -T4 <TARGET_IP>` | Scans all 65535 TCP ports, assumes target is online (`-Pn`), and uses faster timing (`-T4`). Useful for initial reconnaissance. |
| SYN Scan + Service + OS | Stealth scan, service/version & OS detection | `sudo nmap -sS -sV -O --osscan-guess -p- -T4 <TARGET_IP>` | Performs a stealth SYN scan (`-sS`) to detect open ports without full handshake. `-sV` detects service versions. `-O` + `--osscan-guess` identifies OS. Good for detailed footprinting. |
| UDP Scan (Targeted) | Scan common UDP services (DNS, DHCP, NTP) | `sudo nmap -sU -p 53,67,68,69,123,161 -T3 <TARGET_IP>` | UDP scan (`-sU`) checks selected common ports. Slower than TCP; `-T3` balances speed & reliability. Useful to find UDP-based services often missed in TCP scans. |
| Detailed Service Version | Get max version details on specific ports | `sudo nmap -sV --version-intensity 5 -p <open_ports> <TARGET_IP>` | Scans specific ports and collects detailed service/version info. `--version-intensity 5` increases accuracy but takes longer. Ideal for planning vulnerability assessments. |

---

## Observations
- `-Pn` is useful when ICMP/ping is blocked; avoids skipping targets.  
- SYN scans are stealthier than full TCP scans; reduce detection chances.  
- UDP scans are slow but essential for DNS, SNMP, DHCP, and NTP services.  
- Version detection intensity improves accuracy for service identification.  
- OS detection helps in footprinting and attack planning.  

---

## Sample Scan Results Table

| Port | Protocol | Service | State | OS / Version | Notes |
|------|----------|---------|-------|--------------|-------|
| 22   | TCP      | SSH     | Open  | OpenSSH 7.x  | Stealthy access possible; check for weak credentials |
| 80   | TCP      | HTTP    | Open  | Apache 2.4   | Web service; can scan for vulnerabilities |
| 53   | UDP      | DNS     | Open  | ISC BIND     | UDP service; might be exploited in DNS attacks |
| 445  | TCP      | SMB     | Open  | Samba 4.x    | File sharing service; check for misconfigurations |



---

## Learning Outcomes
- Learned different Nmap scan types: TCP full, SYN stealth, UDP, and version detection.  
- Understood scan options for speed, stealth, and accuracy.  
- Learned how to document scan results for reports or GitHub.  
- Practiced planning further assessments based on scan findings.  

---


