
# Packet Analysis – Metasploitable2 Lab

## Author
Aman Jiwani  

## Lab Setup
- **Attacker VM:** Kali Linux  
- **Target VM:** Metasploitable2  
- **Network:** Host-only  

## Tool
- **Wireshark**  
- **Date:** 24 Sep 2025  
- **Target IP:** 192.168.56.104  

---

## Summary of Activities

| Activity | Purpose | Filter / Command |
|----------|---------|----------------|
| Capture HTTP Traffic | Shows unencrypted HTTP traffic | `http` |
| Capture FTP Credentials | Demonstrates plaintext credentials | `ftp`, `frame contains "USER"`, `frame contains "PASS"` |
| Capture DNS Traffic | Reveals visited domains for reconnaissance | `dns` |
| Simulate SYN Flood Attack | Demonstrates DoS via SYN packets | `tcp.flags.syn==1 && tcp.flags.ack==0` |

---

## Observations & Learning
- HTTP/FTP traffic can leak sensitive information if unencrypted.  
- DNS traffic is useful for footprinting.  
- SYN flood attacks can overwhelm services and are detectable in Wireshark.  
- Using filters helps isolate relevant packets for analysis.  

---

## Screenshots
- `screenshots/http_get.png`  
- `screenshots/ftp_credentials.png`  
- `screenshots/dns_query.png`  
- `screenshots/syn_flood_packets.png`  
- `screenshots/netstat_syn_flood.png`  

---

## References
- [Wireshark Documentation](https://www.wireshark.org/docs/)  
- [Metasploitable2 Lab](https://sourceforge.net/projects/metasploitable/)
