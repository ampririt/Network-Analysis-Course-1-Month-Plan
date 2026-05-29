## Session 3 — Network Services: DHCP, DNS & ARP

- [Session 3 — Network Services: DHCP, DNS \& ARP](#session-3--network-services-dhcp-dns--arp)
  - [📖 Lecture](#-lecture)
  - [🛠️ Hands-on Lab](#️-hands-on-lab)
  - [📚 Homework](#-homework)


**Learning Objectives**: Understand how devices obtain IPs, resolve names, and discover MAC addresses. Analyze these protocols in captures.

### 📖 Lecture
- **ARP**: Address Resolution Protocol — MAC discovery, ARP table, gratuitous ARP
- **DHCP**: Discover → Offer → Request → Acknowledge (DORA process)
- **DNS**: Queries & responses, record types (A, AAAA, CNAME, MX, TXT), recursive vs. iterative
- **NAT**: Network Address Translation — how private IPs reach the internet
- Live demo: Capture a DHCP lease renewal in Wireshark

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

### 📚 Homework
- Kurose & Ross Wireshark Lab: **DNS**
- Research: What is DNS poisoning/spoofing? Write a 1-paragraph summary
