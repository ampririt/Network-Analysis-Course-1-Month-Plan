# 📡 Network Analysis Course — 1-Month Plan

> **Duration**: 4 Weeks | **Schedule**: 2 Classes/Week | **Class Length**: 1 Hour 30 Minutes  
> **Total Sessions**: 8 | **Total Contact Hours**: 12 Hours

---

## Course Overview

This course introduces students to the fundamentals of network analysis — from understanding how data flows across networks to capturing, inspecting, and troubleshooting real traffic. Students will gain hands-on experience with industry-standard tools (Wireshark, Cisco Packet Tracer) and develop practical skills applicable to IT support, network administration, and cybersecurity careers.

## Prerequisites

- Basic computer literacy (file management, installing software)
- Conceptual understanding of the internet
- Command-line familiarity is helpful but **not required**

## Required Software (Free)

| Tool | Purpose | Download |
|------|---------|----------|
| **Wireshark** | Live packet capture & protocol analysis | [wireshark.org](https://www.wireshark.org) |
| **Cisco Packet Tracer** | Network simulation & topology design | [netacad.com](https://www.netacad.com/cisco-packet-tracer) |

## Recommended Supplementary Tools

| Tool | Purpose | When Introduced |
|------|---------|-----------------|
| **tcpdump / tshark** | CLI-based packet capture | Session 8 |
| **Nmap** | Network scanning & discovery | Session 7 |
| **NetworkMiner** | Network forensics & file extraction | Session 7 |
| **Scapy (Python)** | Custom packet crafting | Optional / Advanced |

---

## Time Allocation Per Session (90 Minutes)

| Segment | Duration | Description |
|---------|----------|-------------|
| 🔄 Warm-up & Review | 5 min | Recap previous session, answer questions |
| 📖 Lecture & Live Demo | 30–35 min | Core concepts with real-time demonstrations |
| 🛠️ Hands-on Lab | 40–45 min | Guided exercises using Wireshark & Packet Tracer |
| 🎯 Workshop Activity | 10–15 min | Challenge, group exercise, or mini-CTF |
| 📝 Wrap-up & Preview | 5 min | Key takeaways, homework, next session preview |

---

# Week 1: Foundations

---

## Session 1 — Introduction to Network Analysis & The OSI Model

**Learning Objectives**: Understand what network analysis is and why it matters. Learn the OSI model and TCP/IP stack. Set up tools.

### 📖 Lecture (30 min)
- What is network analysis? Real-world use cases (troubleshooting, security, performance)
- The OSI 7-Layer Model — purpose of each layer
- TCP/IP 4-Layer Model — mapping to OSI
- Encapsulation & de-encapsulation: how data becomes frames
- Brief overview: What is a packet? What is a protocol?

### 🛠️ Hands-on Lab (45 min)

**Lab A — Wireshark: First Capture (20 min)**
1. Install and open Wireshark
2. Select network interface and start capturing
3. Open a web browser → visit `http://example.com`
4. Stop capture — observe the packet list
5. Explore the 3 panes: Packet List, Packet Details, Packet Bytes
6. Identify Ethernet, IP, TCP, and HTTP layers in a single packet
7. Practice basic display filters: `http`, `dns`, `icmp`

**Lab B — Cisco Packet Tracer: Build Your First Network (25 min)**
1. Install Packet Tracer and create an account
2. Place 2 PCs and 1 switch on the workspace
3. Connect devices with copper straight-through cables
4. Assign static IP addresses (e.g., `192.168.1.10` and `192.168.1.20`)
5. Use **Simulation Mode** → send a ping → watch packets travel layer-by-layer
6. Observe ARP request/reply and ICMP echo/reply in the event list

### 🎯 Workshop Activity: "Layer Matching Game" (10 min)
- Students receive cards with protocol names (HTTP, TCP, IP, Ethernet, etc.)
- Match each protocol to its correct OSI layer
- Discuss: *"Why does layering matter for network analysis?"*

### 📚 Homework
- Read: Wireshark User Guide — Chapter 1 & 3
- Explore: `wiki.wireshark.org/SampleCaptures` — download 1 PCAP and identify 3 protocols

---

## Session 2 — The Protocol Stack Deep Dive: Ethernet, IP, TCP & UDP

**Learning Objectives**: Understand frame/packet/segment structure. Read protocol headers. Differentiate TCP vs. UDP.

### 📖 Lecture (35 min)
- **Ethernet Frame**: Source/Destination MAC, EtherType, payload
- **IPv4 Header**: Source/Destination IP, TTL, Protocol field, fragmentation
- **TCP Header**: Ports, sequence/ack numbers, flags (SYN, ACK, FIN, RST), 3-way handshake
- **UDP Header**: Ports, length, checksum — connectionless simplicity
- Ports & well-known services: 80 (HTTP), 443 (HTTPS), 53 (DNS), 22 (SSH)
- Live demo: Dissect a single HTTP packet in Wireshark, field by field

### 🛠️ Hands-on Lab (40 min)

**Lab A — Wireshark: TCP 3-Way Handshake (20 min)**
1. Capture traffic while visiting a website
2. Apply filter: `tcp.flags.syn == 1`
3. Identify the SYN → SYN-ACK → ACK sequence
4. Right-click → **Follow TCP Stream** — observe the full conversation
5. Compare a TCP connection (HTTP) vs. a UDP conversation (DNS: `udp.port == 53`)

**Lab B — Cisco Packet Tracer: Simulation Mode Protocol Walk (20 min)**
1. Build a network: 2 PCs → Switch → Router → Server
2. Configure static IPs on different subnets with the router as gateway
3. Switch to **Simulation Mode**
4. Send a ping from PC1 to the Server
5. Step through each packet — observe:
   - ARP resolution at each hop
   - ICMP encapsulation inside IP inside Ethernet
6. Click on each PDU to inspect headers at every layer

### 🎯 Workshop Activity: "Header Hunt" Challenge (15 min)
- Provide a pre-made PCAP file with mixed traffic
- Students race to answer:
  - *"What is the TTL of packet #42?"*
  - *"What port is the server using?"*
  - *"Find a TCP RST — what caused the connection reset?"*
- First to answer all correctly wins

### 📚 Homework
- Kurose & Ross Wireshark Lab: **TCP** (download from textbook website)
- In Packet Tracer: Add a 2nd subnet and configure static routing

---

# Week 2: Network Services & Applications

---

## Session 3 — Network Services: DHCP, DNS & ARP

**Learning Objectives**: Understand how devices obtain IPs, resolve names, and discover MAC addresses. Analyze these protocols in captures.

### 📖 Lecture (30 min)
- **ARP**: Address Resolution Protocol — MAC discovery, ARP table, gratuitous ARP
- **DHCP**: Discover → Offer → Request → Acknowledge (DORA process)
- **DNS**: Queries & responses, record types (A, AAAA, CNAME, MX, TXT), recursive vs. iterative
- **NAT**: Network Address Translation — how private IPs reach the internet
- Live demo: Capture a DHCP lease renewal in Wireshark

### 🛠️ Hands-on Lab (45 min)

**Lab A — Wireshark: DHCP & DNS Analysis (20 min)**
1. Release and renew IP: `ipconfig /release` → `ipconfig /renew` (Windows) or `sudo dhclient -r && sudo dhclient` (Linux/Mac)
2. Capture during renewal — filter: `dhcp` or `bootp`
3. Identify all 4 DORA packets — inspect option fields (lease time, gateway, DNS server)
4. Filter: `dns` → find a query for `google.com`
5. Inspect the DNS response — what IP was returned? What record type?

**Lab B — Cisco Packet Tracer: DHCP & DNS Server Setup (25 min)**
1. Build topology: 3 PCs → Switch → Router → DHCP Server + DNS Server
2. Configure the router with a DHCP pool (network, default-router, dns-server)
3. Set PCs to obtain IP via DHCP
4. Configure DNS Server with entries (e.g., `www.lab.com` → `192.168.1.100`)
5. Switch to **Simulation Mode** — observe DORA packets in real time
6. From a PC, use the web browser to visit `www.lab.com` — observe DNS query then HTTP

### 🎯 Workshop Activity: "DNS Detective" (10 min)
- Provide a PCAP with suspicious DNS traffic
- Students must find:
  - An unusually long DNS query (possible DNS tunneling)
  - A DNS response pointing to an unexpected IP
  - How many unique domains were queried
- Discuss: *"How can DNS be abused by attackers?"*

### 📚 Homework
- Kurose & Ross Wireshark Lab: **DNS**
- Research: What is DNS poisoning/spoofing? Write a 1-paragraph summary

---

## Session 4 — Application Layer Protocols: HTTP, HTTPS & FTP

**Learning Objectives**: Analyze web traffic. Understand plaintext vs. encrypted protocols. Extract data from HTTP sessions.

### 📖 Lecture (30 min)
- **HTTP**: Methods (GET, POST), status codes (200, 301, 404, 500), headers, cookies
- **HTTPS/TLS**: Why encryption matters, TLS handshake overview, certificate inspection
- **FTP**: Control vs. data channel, plaintext credential risk
- Plaintext vs. encrypted traffic — what analysts can and cannot see
- Live demo: Compare HTTP vs. HTTPS capture in Wireshark

### 🛠️ Hands-on Lab (45 min)

**Lab A — Wireshark: HTTP Deep Dive (25 min)**
1. Open the Kurose & Ross HTTP PCAP (or capture from `http://httpbin.org`)
2. Filter: `http.request.method == "GET"` → inspect request headers
3. Right-click → **Follow HTTP Stream** — read the full request/response
4. Find: User-Agent, Content-Type, Set-Cookie headers
5. **Credential Extraction Exercise**: Open provided PCAP with FTP login
   - Filter: `ftp` → find `USER` and `PASS` commands
   - Discuss: *Why is plaintext authentication dangerous?*
6. Attempt the same with HTTPS — observe that payload is encrypted

**Lab B — Cisco Packet Tracer: Web & FTP Server (20 min)**
1. Add a Server to existing topology → enable HTTP and FTP services
2. Customize the web page content on the server
3. From a PC, open the web browser → navigate to the server IP
4. Switch to **Simulation Mode** → observe:
   - TCP 3-way handshake
   - HTTP GET request and response
5. Use the PC's command prompt → `ftp [server IP]` → log in → transfer a file
6. Observe FTP control channel in simulation mode

### 🎯 Workshop Activity: "File Carving Challenge" (15 min)
- Provide a PCAP containing an image transferred over HTTP
- Students must:
  1. Find the HTTP transfer using `http.content_type contains "image"`
  2. **File → Export Objects → HTTP** → save the image
  3. First student to recover and display the image wins
- Bonus: Find a hidden text file transferred via FTP in the same PCAP

### 📚 Homework
- Kurose & Ross Wireshark Lab: **HTTP**
- Capture your own web browsing traffic for 5 minutes → identify 3 different websites visited and the HTTP status codes returned

---

# Week 3: Infrastructure & Troubleshooting

---

## Session 5 — Routing, Switching & VLANs

**Learning Objectives**: Understand how routers forward traffic. Configure VLANs. Analyze routing in captures.

### 📖 Lecture (35 min)
- **Switching**: MAC address table, broadcast vs. unicast, switch forwarding
- **VLANs**: Why segment networks? VLAN tagging (802.1Q), trunk vs. access ports
- **Routing**: Routing table, static vs. dynamic routing, default gateway
- **Inter-VLAN Routing**: Router-on-a-Stick concept, sub-interfaces
- **Subnetting Refresher**: Network/host portions, subnet mask, CIDR notation
- Live demo: `show ip route`, `show vlan brief` in Packet Tracer

### 🛠️ Hands-on Lab (45 min)

**Lab — Cisco Packet Tracer: "Small Office Network" Mega-Lab (45 min)**

> [!IMPORTANT]
> This is the core infrastructure lab of the course. Students build a realistic office network from scratch.

**Topology**: 6 PCs → 2 Switches → 1 Router → 1 Server

1. **Create 3 VLANs** on both switches:
   - VLAN 10: Admin (2 PCs)
   - VLAN 20: Sales (2 PCs)
   - VLAN 30: IT (2 PCs)
2. **Assign switch ports** to VLANs (access mode)
3. **Configure trunk port** between switches
4. **Configure Router-on-a-Stick**:
   - Sub-interfaces: `G0/0.10`, `G0/0.20`, `G0/0.30`
   - Assign gateway IPs for each VLAN
   - Enable `encapsulation dot1Q [vlan-id]`
5. **Configure DHCP** on router for each VLAN
6. **Test inter-VLAN connectivity**: Ping from Admin PC to Sales PC
7. **Verify** with: `show vlan brief`, `show ip interface brief`, `show ip route`

### 🎯 Workshop Activity: "Subnet Design Challenge" (10 min)
- Given: A company has 4 departments with 30, 60, 10, and 120 hosts
- Challenge: Design a subnetting scheme using `172.16.0.0/16`
- Each team presents their subnet plan and explains their choices
- Discuss: efficiency, scalability, and broadcast domain size

### 📚 Homework
- Save the Small Office .pkt file — it will be used in Session 6
- Research: What is OSPF and how does it differ from static routing?

---

## Session 6 — Network Troubleshooting Methodology

**Learning Objectives**: Apply systematic troubleshooting. Use CLI tools. Diagnose issues from packet captures and simulations.

### 📖 Lecture (30 min)
- **Troubleshooting Methodology**: Identify → Establish theory → Test → Establish plan → Verify → Document
- **Bottom-up approach**: Physical → Data Link → Network → Transport → Application
- **Essential CLI Tools**:
  - `ping` — connectivity test (ICMP)
  - `traceroute` / `tracert` — path discovery
  - `nslookup` / `dig` — DNS verification
  - `ipconfig` / `ifconfig` — interface status
  - `arp -a` — ARP table inspection
  - `netstat` — active connections & ports
- **Common Issues**: Wrong subnet mask, missing default gateway, DNS failure, duplex mismatch, VLAN misconfiguration
- Live demo: Diagnose a "no internet" scenario step by step

### 🛠️ Hands-on Lab (40 min)

**Lab A — Cisco Packet Tracer: "The Broken Network" (25 min)**

> [!TIP]
> Pre-built `.pkt` files with intentional misconfigurations are extremely effective for teaching troubleshooting.

Students receive a pre-configured network with **5 hidden problems**:
1. ❌ A PC has the wrong default gateway
2. ❌ A switch port is in the wrong VLAN
3. ❌ The trunk link is configured as access mode
4. ❌ A router sub-interface has the wrong IP
5. ❌ The DHCP pool has an incorrect DNS server address

Students must:
- Use `ping`, `show` commands, and simulation mode to find all 5 issues
- Fix each issue and verify end-to-end connectivity
- Document each problem and solution

**Lab B — Wireshark: Troubleshooting from PCAPs (15 min)**
1. Open a PCAP of a failed web connection
2. Identify the issue:
   - Is there a TCP SYN with no SYN-ACK? → Server down or firewall
   - Is there DNS failure? → Filter `dns` and check for errors
   - Is there a TCP RST? → Port closed or connection refused
3. Apply filters: `tcp.analysis.retransmission`, `tcp.flags.reset == 1`, `dns.flags.rcode != 0`
4. Write a brief "troubleshooting report" with findings

### 🎯 Workshop Activity: "Troubleshooting Relay Race" (15 min)
- Teams of 3–4 students
- Each team receives the same broken Packet Tracer network
- Team members take turns — each person fixes ONE issue then passes to the next
- First team with full connectivity wins
- Debrief: Discuss the different approaches teams used

### 📚 Homework
- Practice: Create your own "broken network" in Packet Tracer (minimum 3 problems) — bring the `.pkt` file to share in Session 7
- Capture traffic while running `traceroute google.com` — how many hops?

---

# Week 4: Security & Capstone

---

## Session 7 — Network Security & Threat Detection

**Learning Objectives**: Recognize common network attacks in captures. Understand IDS/IPS concepts. Perform basic network forensics.

### 📖 Lecture (35 min)
- **Network Attack Types**:
  - Reconnaissance: Port scanning (SYN scan, connect scan)
  - Man-in-the-Middle: ARP spoofing, DNS poisoning
  - Denial of Service: SYN flood, amplification attacks
  - Data exfiltration: DNS tunneling, covert channels
- **Defense Mechanisms**: Firewalls, IDS/IPS (Snort, Suricata), ACLs, port security
- **Intro to Nmap**: Basic scan types and their signatures
- **Wireshark Security Filters**:
  - SYN scan detection: `tcp.flags.syn == 1 && tcp.flags.ack == 0`
  - ARP anomalies: `arp.duplicate-address-detected`
  - DNS tunneling indicators: unusually long queries, high TXT record volume
- Live demo: Show an Nmap SYN scan in Wireshark

### 🛠️ Hands-on Lab (40 min)

**Lab A — Wireshark: "Spot the Attack" (25 min)**
1. **Scan Detection**: Open a PCAP containing an Nmap SYN scan
   - Filter: `tcp.flags.syn == 1 && tcp.flags.ack == 0`
   - Identify the scanning host and target
   - Use **Statistics → Conversations** to see which ports were probed
   - Determine: Was it a SYN scan, connect scan, or something else?

2. **Rogue Device Hunt**: Open a PCAP from a compromised network
   - Use **Statistics → Endpoints** to list all active hosts
   - Identify the host with abnormal traffic volume or unusual protocol usage
   - Check for ARP anomalies (duplicate IPs, gratuitous ARP from unexpected MAC)

3. **Credential Theft Analysis**: Open a PCAP with Telnet/FTP sessions
   - Follow the TCP stream → extract plaintext username and password
   - Discuss: *How would SSH/SFTP prevent this?*

**Lab B — Cisco Packet Tracer: ACLs & Port Security (15 min)**
1. Open the Small Office network from Session 5
2. Configure a **Standard ACL** to block VLAN 20 (Sales) from accessing the Server
3. Apply the ACL to the router sub-interface
4. Test: Verify Sales PCs cannot reach the server but Admin/IT can
5. Configure **Port Security** on a switch port — set max MAC addresses to 1

### 🎯 Workshop Activity: "Mini CTF — Network Forensics" (15 min)
- Teams receive a PCAP from a simulated breach scenario
- **5 Flags to find**:
  1. 🚩 What IP performed the port scan?
  2. 🚩 What service was exploited (find the port)?
  3. 🚩 Extract the attacker's login credentials
  4. 🚩 What file was exfiltrated (export from HTTP objects)?
  5. 🚩 What is the hidden message inside the file?
- Scoring: 20 points per flag, bonus for speed

### 📚 Homework
- Prepare for Capstone: Review all sessions
- Optional: Try **TryHackMe** room *"Wireshark: The Basics"* (free)

---

## Session 8 — Capstone Project & Course Review

**Learning Objectives**: Apply all skills in a comprehensive scenario. Demonstrate proficiency. Explore career paths.

### 📖 Lecture & Review (25 min)
- Quick review: OSI model → Protocols → Services → Routing → Troubleshooting → Security
- **Beyond Wireshark**:
  - `tcpdump` — capture on headless servers: `tcpdump -i eth0 -w capture.pcap`
  - `tshark` — CLI Wireshark: `tshark -r file.pcap -Y "http.request"`
- **Career Paths & Certifications**:
  - CompTIA Network+ (N10-009)
  - CompTIA Security+ (SY0-701)
  - GIAC Certified Intrusion Analyst (GCIA)
  - Cisco CCNA
- **Continuing Education Resources**:
  - TryHackMe, Blue Team Labs Online, CyberDefenders
  - Malware-Traffic-Analysis.net (advanced PCAPs)
  - Chris Greer's YouTube (Wireshark tutorials)

### 🛠️ Capstone Challenge (50 min)

> [!IMPORTANT]
> The capstone is a practical, scenario-based assessment that covers the entire course.

**Scenario**: *"A company reports that employees cannot access the internal website, and suspicious traffic has been detected on the network."*

**Part 1 — Packet Tracer: Fix the Network (20 min)**
- Students receive a broken network topology
- Must restore connectivity by fixing routing, VLAN, and DHCP issues
- Must implement one ACL as a security measure

**Part 2 — Wireshark: Incident Analysis (20 min)**
- Students receive a PCAP from the "compromised" network
- Must answer:
  1. What services are running on the network?
  2. Identify the attacker's IP address
  3. What type of scan was performed?
  4. Were any credentials compromised?
  5. Was any data exfiltrated? If so, recover it.
- Write a 1-page **Incident Report** with findings and recommendations

**Part 3 — Presentation (10 min)**
- Each student/team presents their top finding to the class
- Peer discussion and feedback

### 🎯 Workshop Activity: "Build & Break" Competition (remaining time)
- **Phase 1**: Each team builds a small network in Packet Tracer (5 min)
- **Phase 2**: Teams swap — each team must find and fix the other team's hidden problems
- Awards for: Best Network Design, Fastest Fixer, Most Creative Problem

---

## Assessment Summary

| Assessment | Weight | Session |
|------------|--------|---------|
| Class Participation & Labs | 30% | Every session |
| Homework Assignments | 20% | Weekly |
| Mini CTF Performance | 15% | Session 7 |
| Capstone Project | 35% | Session 8 |

---

## Recommended Resources

### PCAP Datasets for Practice
| Source | URL | Level |
|--------|-----|-------|
| Wireshark Sample Captures | `wiki.wireshark.org/SampleCaptures` | Beginner |
| Netresec PCAP Repository | `netresec.com` | Intermediate |
| Malware-Traffic-Analysis.net | `malware-traffic-analysis.net` | Advanced |
| Digital Corpora | `digitalcorpora.org` | Intermediate |

### Lab Resources
| Source | URL | Description |
|--------|-----|-------------|
| Kurose & Ross Wireshark Labs | UMass textbook site | Gold-standard protocol labs |
| Jeremy's IT Lab | YouTube + GitHub | 76+ Packet Tracer labs |
| Cisco NetAcad | `netacad.com` | Official structured courses |
| TryHackMe | `tryhackme.com` | Interactive security rooms |
| Blue Team Labs Online | `blueteamlabs.online` | Defensive challenges |

---

## Open Questions

> [!IMPORTANT]
> **Target Audience**: What level are the students? (e.g., university undergrads, IT professionals, complete beginners?) This affects the depth of subnetting, routing, and security content.

> [!IMPORTANT]
> **Language**: Should course materials be prepared in English, Thai, or bilingual?

> [!NOTE]
> **Lab Environment**: Will students have their own laptops with admin rights to install Wireshark and Packet Tracer? Or will a computer lab be used?

> [!NOTE]
> **Delivery Format**: Is this in-person, online (Zoom/Teams), or hybrid? This affects how labs and group activities are conducted.

> [!NOTE]
> **Output Format**: Would you like me to also generate this plan as a PowerPoint (`.pptx`) presentation or a detailed syllabus document?


