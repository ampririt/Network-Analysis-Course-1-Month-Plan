## Session 3 — Network Services& Security

- [Session 3 — Network Services\& Security](#session-3--network-services-security)
  - [Lecture](#lecture)
  - [Hands-on Lab](#hands-on-lab)
  - [Homework](#homework)


**Learning Objectives**: Understand how devices obtain IPs, resolve names, and discover MAC addresses. Analyze these protocols in captures.

### Lecture

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

### Hands-on Lab

> [!TIP]
> Each lab below is a short introduction. **Full step-by-step instructions, diagrams, and lab questions live in the companion guides** — follow those pages while you work.

---

**Lab A — Wireshark: DHCP & DNS Analysis (~20 min)**

Watch a device get an address and resolve a name. You'll release/renew your IP, capture the four **DHCP DORA** packets (`Discover → Offer → Request → ACK`) and read the lease/gateway/DNS options, then filter `dns` to inspect a **query and its response** — identifying the IP returned and the record type (A / AAAA / CNAME).

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Wireshark — Lab A: DHCP & DNS Analysis](./WIRESHARK_GUIDE.md#lab-a--dhcp--dns-analysis)**

---

**Lab B — Cisco Packet Tracer: Secure Management & File Transfer (SSH & FTP)**

Reuse the **multi-subnet network from Session 2** and add two application-layer services. You'll harden the router for **SSH** remote management (generate an RSA key, restrict the vty lines to SSH so Telnet is refused), stand up an **FTP** server on the Management subnet, transfer a file from a PC across the router, and see why SSH is safe to sniff but plaintext FTP is not.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Packet Tracer — Lab B: Secure Management & File Transfer (SSH & FTP)](./PACKET_TRACER_GUIDE.md)**

---

**Lab C — Cisco Packet Tracer: DHCP & DNS (continuing from Lab B)**

**Continue from your Lab B file.** Make the same multi-subnet network self-configuring: add a **DNS** service to the FTP-server (`ftp.lab.local → 192.168.0.98`), configure **per-department DHCP pools** on the router, switch the PCs from static to **DHCP**, then reach the server **by name** — running the server side of the very DORA/DNS exchanges you captured in Lab A.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Packet Tracer — Lab C: DHCP & DNS, continuing from Lab B](./PACKET_TRACER_GUIDE.md#lab-c--dhcp--dns-continuing-from-lab-b)**

### Homework
- Kurose & Ross Wireshark Lab: **DNS**
- Research: What is DNS poisoning/spoofing? Write a 1-paragraph summary
