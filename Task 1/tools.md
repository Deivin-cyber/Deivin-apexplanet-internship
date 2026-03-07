Wireshark

What it is:
Wireshark is a network analysis tool used to capture and examine data packets moving through a network.

How it works:
It puts the network interface into promiscuous mode, which allows it to capture all network traffic, not only the traffic meant for the computer.

Use cases:
It is used to troubleshoot network issues, study how applications communicate, and detect possible security problems.

Key features:

Packet List: Displays all captured packets in real time.

Filters: Capture filters limit what packets are recorded, and display filters help focus on specific packets after capturing.

Nmap

What it is:
Nmap (Network Mapper) is a tool used for network discovery and security scanning.

How it works:
It sends special packets to a target system and studies the responses to identify active hosts, open ports, services, and operating systems.

Use cases:
It helps create a map of a network, find open ports, and check systems for possible vulnerabilities.

Key commands:

nmap [target] – Performs a basic scan of a host.

nmap -sS – Performs a TCP SYN (stealth) scan.

nmap -sV – Detects service versions running on ports.

nmap -p [port] – Scans a specific port or range of ports.

Burp Suite

What it is:
Burp Suite is a tool used for testing the security of web applications.

How it works:
It works as an interception proxy between the browser and the internet, allowing users to capture and modify HTTP or HTTPS requests and responses.

Use cases:
It helps find web security problems such as SQL injection and Cross-Site Scripting (XSS) and is used for manual security testing.

Netcat

What it is:
Netcat (nc) is a command-line networking tool often called the “Swiss Army knife” of networking because it can perform many tasks.

How it works:
It can work as a client to connect to another system or as a server to listen for incoming connections.

Use cases:
It is used for port scanning, transferring files, creating simple backdoors, and debugging network connections.

Key options:

-v – Verbose mode (shows detailed output).

-z – Scan mode without sending data.

-l – Listen mode for incoming connections.

-p – Specifies the port number.
