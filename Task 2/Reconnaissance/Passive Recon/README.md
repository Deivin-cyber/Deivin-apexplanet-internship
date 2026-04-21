 
🔍 Active Reconnaissance

📌 Objective

Active reconnaissance involves directly interacting with the target system to identify live hosts and gather service-level information.

---

🧠 1. Ping Sweep (Host Discovery)

A ping sweep was performed to identify active hosts in the network:

nmap -sn 192.168.126.0/24

🔎 Result:

The scan detected active devices within the subnet, including:

- 192.168.126.129 (Target – Metasploitable2)

📖 Analysis:

- The "-sn" option disables port scanning and only checks for host availability.
- This helps identify which systems are alive before performing deeper scans.
- It reduces unnecessary scanning noise and improves efficiency.

---

🧾 2. Banner Grabbing

Banner grabbing was used to collect information about running services.

🔧 FTP Banner (Port 21)

nc 192.168.126.129 21

🔎 Result:

The FTP service responded with a banner revealing server information (e.g., vsFTPd version).

---

📖 Analysis:

- Banner grabbing helps identify:
  - Service type
  - Software version
- This information is critical for:
  - Detecting outdated services
  - Mapping known vulnerabilities

---

⚠️ Key Observations

- The target system exposes multiple services publicly.
- FTP service allows direct interaction and reveals version details.
- Such exposure increases the attack surface.

---

✅ Conclusion

Active reconnaissance successfully identified:

- Live hosts in the network using ping sweep
- Service-level information through banner grabbing

These findings provide a strong foundation for further port scanning and vulnerability analysis.
