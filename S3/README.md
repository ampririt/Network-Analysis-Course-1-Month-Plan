## Session 3 — Network Services& Security

- [Session 3 — Network Services\& Security](#session-3--network-services-security)
  - [📖 Lecture](#-lecture)
  - [🛠️ Hands-on Lab](#️-hands-on-lab)
    - [Wireshark — Labs A & C guide →](./WIRESHARK_GUIDE.md)
    - [Packet Tracer — Lab B guide →](./PACKET_TRACER_GUIDE.md)
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

> [!TIP]
> Each lab below is a short introduction. **Full step-by-step instructions, diagrams, and lab questions live in the companion guides** — follow those pages while you work.

---

**Lab A — Wireshark: DHCP & DNS Analysis (~20 min)**

Watch a device get an address and resolve a name. You'll release/renew your IP, capture the four **DHCP DORA** packets (`Discover → Offer → Request → ACK`) and read the lease/gateway/DNS options, then filter `dns` to inspect a **query and its response** — identifying the IP returned and the record type (A / AAAA / CNAME).

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Wireshark — Lab A: DHCP & DNS Analysis](./WIRESHARK_GUIDE.md#-lab-a--dhcp--dns-analysis)**

---

**Lab B — Cisco Packet Tracer: DHCP & DNS Server Setup (~25 min)**

Build the **server side** of those services. You'll configure a router **DHCP pool** (network, default-router, dns-server), set PCs to obtain addresses automatically, add a **DNS A record** (`www.lab.com` → `192.168.1.100`), watch **DORA** unfold in Simulation Mode, and finally browse to the site **by name** from a PC.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Packet Tracer — Lab B: DHCP & DNS Server Setup](./PACKET_TRACER_GUIDE.md)**

---

**Lab C — Wireshark: Network Security Analysis (~25 min)**

> ⚠️ Run only on your own lab network / a host you control. Scanning or spoofing networks you don't own may be illegal.

Learn to recognise attack *signatures* in a capture: spot **ARP spoofing** (two MACs claiming one IP), identify a **port scan** (a burst of SYNs to sequential ports, open vs. closed via SYN-ACK/RST), and compare **plaintext HTTP vs. encrypted TLS** to see why HTTPS matters — then map each attack to the defense that stops it.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Wireshark — Lab C: Network Security Analysis](./WIRESHARK_GUIDE.md#-lab-c--network-security-analysis)**

### 📚 Homework
- Kurose & Ross Wireshark Lab: **DNS**
- Research: What is DNS poisoning/spoofing? Write a 1-paragraph summary
