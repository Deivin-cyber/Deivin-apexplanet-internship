
🔍 Active Reconnaissance

📌 Objective

The goal of active reconnaissance is to directly interact with the target system to identify live hosts, open ports, running services, and potential entry points for further analysis.

The target system used in this lab:

- Target IP: 192.168.126.129
- Attacker Machine: Kali Linux

---

🧠 1. Host Discovery (Ping Sweep)

To identify active devices in the network, a ping sweep was performed:

nmap -sn 192.168.126.0/24

🔎 Result:

The scan identified active hosts within the subnet, including the target system:

- 192.168.126.129 (Metasploitable2)

📖 Analysis:

This step helps in mapping the network and identifying reachable systems before performing deeper scans.

---

🚪 2. Basic Port Scanning

A basic scan was performed to identify open ports:

nmap 192.168.126.129

🔎 Result:

The scan revealed multiple open ports such as:

- 21 (FTP)
- 22 (SSH)
- 23 (Telnet)
- 80 (HTTP)

📖 Analysis:

Open ports indicate active services running on the target system. These services can potentially be exploited if misconfigured or outdated.

---

⚙️ 3. Service & Version Detection

To gather detailed information about services and their versions:

nmap -sV 192.168.126.129

🔎 Result:

Service versions such as FTP server version, SSH version, and web server details were identified.

📖 Analysis:

Service version detection helps identify outdated or vulnerable software that can be targeted in later stages.

---

🖥️ 4. OS Detection

To identify the operating system of the target:

nmap -O 192.168.126.129

🔎 Result:

The system was identified as a Linux-based machine.

📖 Analysis:

Knowing the operating system allows attackers to choose appropriate exploits and tools.

---

📡 5. Full Scan (Comprehensive Analysis)

A full scan combining multiple techniques was executed:

nmap -sS -sV -O -p- 192.168.126.129

🔎 Result:

- All TCP ports scanned
- Multiple services detected
- OS fingerprinting completed

📖 Analysis:

This scan provides a complete overview of the target system, including all open ports, running services, and OS details.

---

🔐 6. Banner Grabbing

Manual banner grabbing was performed to extract service information:

nc 192.168.126.129 21

🔎 Result:

The FTP service returned a banner indicating the server type and version.

📖 Analysis:

Banner information can reveal software versions, which may contain known vulnerabilities.

---

⚠️ Overall Findings

- Multiple open ports indicate a poorly secured system.
- Presence of services like FTP and Telnet suggests insecure configurations.
- Service versions may contain known vulnerabilities.
- The system is highly suitable for vulnerability scanning and exploitation.

---

✅ Conclusion

Active reconnaissance successfully identified:

- Live hosts in the network
- Open ports and services
- Service versions and OS details

This information forms the foundation for the next phase: Vulnerability Scanning and Exploitation.
