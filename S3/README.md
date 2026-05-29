## Session 3 — Network Services& Security

- [Session 3 — Network Services\& Security](#session-3--network-services-security)
  - [📖 Lecture](#-lecture)
  - [🛠️ Hands-on Lab](#️-hands-on-lab)
  - [📚 Homework](#-homework)


**Learning Objectives**: Understand how devices obtain IPs, resolve names, and discover MAC addresses. Analyze these protocols in captures.

### 📖 Lecture

**Network Services**
- A *network service* is a function one device (**server**) provides to others (**clients**) over the network — addressed by **IP + transport protocol + port** (e.g., DNS = `udp/53`, DHCP = `udp/67/68`).
- They sit on top of the protocol stack from Sessions 1–2 and are the backbone of "the network working": a device joining a LAN gets an address (**DHCP**), finds the next hop's MAC (**ARP**), resolves names (**DNS**), and reaches the internet (**NAT**).
- **ARP**: Address Resolution Protocol — MAC discovery, ARP table, gratuitous ARP
- **DHCP**: Discover → Offer → Request → Acknowledge (DORA process)
- **DNS**: Queries & responses, record types (A, AAAA, CNAME, MX, TXT), recursive vs. iterative
- **ICMP**: Echo request/reply (`ping`), TTL-exceeded & unreachable messages, traceroute mechanics
- **NAT**: Network Address Translation — how private IPs reach the internet
- **Double NAT**: Stacked NAT (e.g., ISP router behind your own router) — symptoms and why it breaks port forwarding
- **HTTP/HTTPS & TLS**: Application-layer web traffic and encryption — *huge topic, deserves its own session on the application layer*
- **SSH / Telnet**: Remote management — *pairs well with a security session*
- **SMTP / IMAP / POP3**: Email protocols — *only if your audience cares about mail flow*
- Live demo: Capture a DHCP lease renewal in Wireshark

**Network Security**
- **Why services are targets**: Most core services (ARP, DHCP, DNS) have *no built-in authentication*, so they are easy to spoof or abuse.
- **CIA triad**: Confidentiality, Integrity, Availability — the goals every security control protects.
- **Common attacks on services**:
  - **ARP spoofing / poisoning** → Man-in-the-Middle (MITM), traffic interception
  - **Rogue DHCP server** → handing out bad gateway/DNS to redirect victims
  - **DNS spoofing / cache poisoning** → sending users to malicious IPs
  - **MAC flooding / port stealing** → overwhelming a switch's CAM table
  - **DoS / DDoS** → flooding a service to exhaust its resources
- **Reconnaissance & scanning**: Port scans, ping sweeps, OS fingerprinting (e.g., Nmap) — what an attacker does *before* an attack, and what it looks like in a capture.
- **Defenses**: Switch hardening (**DHCP snooping**, **Dynamic ARP Inspection**, **port security**), encryption (**HTTPS/TLS**, **SSH** over Telnet), firewalls & segmentation (**VLANs**, **DMZ**), and **DNSSEC**.
- **The analyst's mindset**: Establish a *baseline* of normal traffic so anomalies (unexpected ARP replies, spikes, odd ports) stand out.

### 🛠️ Hands-on Lab

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

**Lab C — Wireshark: Network Security Analysis (25 min)**

> ⚠️ Run only on your own lab network / a host you control. Scanning or spoofing networks you don't own may be illegal.

1. **Spot an ARP anomaly**:
   - Filter: `arp` — note your normal request/reply pairs and the gateway's MAC.
   - Look for **two different MACs claiming the same IP**, or a flood of gratuitous ARPs → the signature of **ARP spoofing**.
   - Tip: Wireshark's *Analyze → Expert Information* flags "duplicate IP address detected."
2. **Identify a port scan**:
   - On a lab host, run `nmap -sS <target-ip>` from another machine you control.
   - In Wireshark, filter: `tcp.flags.syn == 1 && tcp.flags.ack == 0`.
   - Observe **many SYNs to sequential ports** from one source — classic scan behavior.
   - Compare RST (closed) vs. SYN-ACK (open) responses to see which ports are open.
3. **Inspect plaintext vs. encrypted**:
   - Visit an `http://` site and a `https://` site while capturing.
   - Filter `http` → confirm you can read credentials/content in **Follow TCP Stream**.
   - Filter `tls` → confirm the payload is encrypted (only the handshake/SNI is visible) — *why HTTPS matters*.
4. **Reflect**: Which of these attacks would **DHCP snooping**, **Dynamic ARP Inspection**, or **HTTPS** have prevented?

### 📚 Homework
- Kurose & Ross Wireshark Lab: **DNS**
- Research: What is DNS poisoning/spoofing? Write a 1-paragraph summary
