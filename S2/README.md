## Session 2 — The Protocol Stack Deep Dive: Ethernet, IP, TCP & UDP

- [Session 2 — The Protocol Stack Deep Dive: Ethernet, IP, TCP \& UDP](#session-2--the-protocol-stack-deep-dive-ethernet-ip-tcp--udp)
  - [📖 Lecture](#-lecture)
  - [Break (10 min)](#break-10-min)
  - [🛠️ Hands-on Lab](#️-hands-on-lab)
  - [🔍 Deep Dive: IP Addressing \& Subnetting](#-deep-dive-ip-addressing--subnetting)
    - [IPv4 Address Classes (Legacy — For Context)](#ipv4-address-classes-legacy--for-context)
    - [Private IP Address Ranges (RFC 1918)](#private-ip-address-ranges-rfc-1918)
    - [Subnetting Quick-Reference Table](#subnetting-quick-reference-table)
    - [How to Subnet: Step-by-Step](#how-to-subnet-step-by-step)
  - [📚 Homework](#-homework)


**Learning Objectives**: Understand frame/packet/segment structure. Read protocol headers. Differentiate TCP vs. UDP. Design IP addressing schemes and perform subnetting.

### 📖 Lecture

**Part 1 — Protocol Headers (25 min)**
- **Ethernet Frame**: Source/Destination MAC, EtherType, payload
- **IPv4 Header**: Source/Destination IP, TTL, Protocol field, fragmentation
- **TCP Header**: Ports, sequence/ack numbers, flags (SYN, ACK, FIN, RST), 3-way handshake
- **UDP Header**: Ports, length, checksum — connectionless simplicity
- Ports & well-known services: 80 (HTTP), 443 (HTTPS), 53 (DNS), 22 (SSH)

**Part 2 — IP Network Design & Subnetting (25 min)**
- **IPv4 Address Structure**: Network portion vs. host portion, dotted-decimal & binary representation
- **Subnet Masks**: How masks define network boundaries — `255.255.255.0` (`/24`), `255.255.0.0` (`/16`)
- **CIDR Notation**: Classless Inter-Domain Routing — why classful addressing is obsolete
- **Subnetting**: Borrowing host bits to create subnets — calculating network address, broadcast address, usable host range
- **Private vs. Public IP Ranges**: RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) and why NAT is needed
- **IP Design Best Practices**: Address planning for departments, growth considerations, documentation standards


### Break (10 min)
- Step away, stretch, and grab a drink before diving into the hands-on labs.

### 🛠️ Hands-on Lab

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

**Lab C — Cisco Packet Tracer: IP Network Design & Multi-Subnet Build (40 min)**

> [!IMPORTANT]
> This lab builds the foundation for all future Packet Tracer exercises. Students design an IP scheme from scratch.

**Scenario**: A small company has 3 departments — **Engineering** (50 hosts), **Marketing** (25 hosts), and **Management** (10 hosts). Design and build the network using `192.168.0.0/24`.

1. **Subnetting on Paper (10 min)**:
   * Calculate the required subnet size for each department
   * Determine the subnet mask, network address, broadcast address, and usable range for each:
     - Engineering: `192.168.0.0/26` → 62 usable hosts
     - Marketing: `192.168.0.64/27` → 30 usable hosts
     - Management: `192.168.0.96/28` → 14 usable hosts
   * Document the IP allocation plan in a table
2. **Build in Packet Tracer (15 min)**:
   * Drag onto workspace: 6 PCs (2 per department), 3 Switches, 1 Router
   * Connect PCs to their department switch → connect all switches to the router
   * Configure router sub-interfaces with the gateway IP for each subnet
   * Assign static IPs to each PC within the correct subnet range
3. **Verify Connectivity (10 min)**:
   * Ping within the same subnet → should succeed immediately
   * Ping across subnets → should succeed via the router
   * Deliberately assign a PC the wrong subnet mask → observe and explain the failure
   * Run `show ip interface brief` on the router to verify all sub-interfaces are `up/up`
4. **IP Conflict Demonstration (5 min)**:
   * Assign two PCs the same IP address → observe the conflict warning
   * Discuss: *Why is IP address management (IPAM) critical in real networks?*

---

### 🔍 Deep Dive: IP Addressing & Subnetting

#### IPv4 Address Classes (Legacy — For Context)

| Class | First Octet Range | Default Mask | Network/Host Split | Example |
|:---:|:---:|:---:|:---:|:---:|
| **A** | 1 – 126 | `255.0.0.0` (`/8`) | N.H.H.H | `10.0.0.1` |
| **B** | 128 – 191 | `255.255.0.0` (`/16`) | N.N.H.H | `172.16.0.1` |
| **C** | 192 – 223 | `255.255.255.0` (`/24`) | N.N.N.H | `192.168.1.1` |

> [!NOTE]
> Classful addressing is obsolete. Modern networks use **CIDR** (Classless Inter-Domain Routing) which allows any prefix length. Understanding classes is still useful for recognizing private ranges and default behaviors.

#### Private IP Address Ranges (RFC 1918)

| Range | CIDR | Usable Addresses | Common Use |
|:---:|:---:|:---:|:---:|
| `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` | ~16.7 million | Large enterprises, cloud VPCs |
| `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` | ~1 million | Mid-size organizations |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` | ~65,000 | Home networks, small offices |

#### Subnetting Quick-Reference Table

| CIDR | Subnet Mask | Total IPs | Usable Hosts | Typical Use |
|:---:|:---:|:---:|:---:|:---:|
| `/30` | `255.255.255.252` | 4 | 2 | Point-to-point links |
| `/28` | `255.255.255.240` | 16 | 14 | Small management LANs |
| `/27` | `255.255.255.224` | 32 | 30 | Small departments |
| `/26` | `255.255.255.192` | 64 | 62 | Medium departments |
| `/25` | `255.255.255.128` | 128 | 126 | Large departments |
| `/24` | `255.255.255.0` | 256 | 254 | Standard LAN segment |
| `/23` | `255.255.254.0` | 512 | 510 | Large LAN |
| `/22` | `255.255.252.0` | 1,024 | 1,022 | Campus building |
| `/16` | `255.255.0.0` | 65,536 | 65,534 | Large site |

#### How to Subnet: Step-by-Step

1. **Determine requirements**: How many subnets? How many hosts per subnet?
2. **Choose the base network**: e.g., `192.168.1.0/24`
3. **Borrow bits**: Each borrowed host bit doubles the number of subnets but halves the hosts
   - `/24` → `/25` = 2 subnets of 126 hosts each
   - `/24` → `/26` = 4 subnets of 62 hosts each
   - `/24` → `/27` = 8 subnets of 30 hosts each
4. **Calculate for each subnet**:
   - **Network address**: First IP (all host bits = 0)
   - **Broadcast address**: Last IP (all host bits = 1)
   - **Usable range**: Network + 1 → Broadcast − 1
   - **Gateway**: Typically the first or last usable IP

> [!TIP]
> **Quick formula**: For a `/(24+n)` subnet:
> - Subnet size = `256 / 2^n` addresses
> - Usable hosts = subnet size − 2
> - Subnets start at multiples of the subnet size
>
> Example: `/26` → `n=2` → size = 64 → subnets at `.0`, `.64`, `.128`, `.192`

---

### 📚 Homework
- Kurose & Ross Wireshark Lab: **TCP** (download from textbook website)
- **IP Design Exercise**: Given the network `10.0.0.0/16`, design an IP scheme for 5 departments with 100, 60, 30, 15, and 5 hosts. Document subnet addresses, masks, usable ranges, and gateways in a table
- In Packet Tracer: Build the design above with at least 1 PC per subnet and verify cross-subnet connectivity via a router

---