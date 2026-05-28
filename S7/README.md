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