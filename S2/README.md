## Session 2 — The Protocol Stack Deep Dive: Ethernet, IP, TCP & UDP

> 🌐 **English** | [日本語](./README.ja.md)

- [Session 2 — The Protocol Stack Deep Dive: Ethernet, IP, TCP \& UDP](#session-2--the-protocol-stack-deep-dive-ethernet-ip-tcp--udp)
  - [Lecture](#lecture)
  - [Break (10 min)](#break-10-min)
  - [Hands-on Lab](#hands-on-lab)
  - [Deep Dive: IP Addressing \& Subnetting](#deep-dive-ip-addressing--subnetting)
    - [IPv4 Address Classes (Legacy — For Context)](#ipv4-address-classes-legacy--for-context)
    - [Private IP Address Ranges (RFC 1918)](#private-ip-address-ranges-rfc-1918)
    - [Subnetting Quick-Reference Table](#subnetting-quick-reference-table)
    - [How to Subnet: Step-by-Step](#how-to-subnet-step-by-step)
  - [Homework](#homework)


**Learning Objectives**: Understand frame/packet/segment structure. Read protocol headers. Differentiate TCP vs. UDP. Design IP addressing schemes and perform subnetting.

### Lecture

> The lecture runs in **two halves** — first *reading* the protocols that carry every conversation (headers, field by field), then *designing* the IP space those packets live in. Live, interactive simulations (layer-by-layer delivery, the 3-way handshake, the sliding window, UDP loss, and IP/subnet calculation) run throughout.

**Part 1 — Reading the Protocols**
- **Data units & encapsulation**: how one HTTP request travels *down* the stack (L7 → L1) and back *up* — each layer wraps the one above. The data unit per layer: **segment / datagram** (L4) → **packet** (L3) → **frame** (L2).
- **Ethernet frame (L2)**: Destination MAC + Source MAC (48 bits each), **EtherType** (`0x0800` IPv4, `0x0806` ARP, `0x86DD` IPv6), payload (46–1500 bytes), and the **FCS** CRC trailer.
- **IPv4 header (L3, 20 bytes)**: Version, IHL, DSCP/ECN, Total Length, Identification + Flags (DF/MF) + Fragment Offset (**fragmentation**), **TTL**, **Protocol**, Header Checksum, and Source/Destination IP.
- **TCP header (L4 segment)**: Source/Destination **Port**, **Sequence Number** (makes TCP *ordered*), **Acknowledgement Number** (makes TCP *reliable*), Data Offset, **Flags** (SYN, ACK, FIN, RST), and Window.
- **UDP header (L4 datagram, 8 bytes)**: Source Port, Destination Port, Length, Checksum — and nothing more.
- **TCP vs. UDP**: connection-oriented & reliable (handshake, ACKs, retransmission, flow control) versus connectionless & fast (fire-and-forget). Choose by what failure costs.
- **The 3-way handshake**: `SYN → SYN/ACK → ACK` opens a connection and synchronises each side's sequence numbers.
- **TCP flow control — the sliding window**: the byte stream is sliced into MSS-sized segments; each ACK names the next expected byte and slides the window forward; the receiver's advertised window is its free buffer.
- **Ports & demultiplexing**: one host, one IP, but **65,536 ports** (0–65535) — the destination port picks which program receives the packet. Ranges: **well-known** 0–1023, **registered** 1024–49151, **ephemeral** 49152–65535. Common services: 20/21 FTP, 22 SSH, 25 SMTP, 53 DNS, 67/68 DHCP, 80 HTTP, 443 HTTPS/QUIC, 3389 RDP.

**Part 2 — Designing the IP Space**
- **Address structure**: every IPv4 address is 32 bits / 4 octets, split into a **network** portion and a **host** portion — the subnet mask draws the line (e.g. `192.168.1.10/24`).
- **From classful to classless**: the old Class A/B/C system (`/8`, `/16`, `/24`) wasted huge blocks and bloated routing tables; **CIDR** (1993) lets the network/host boundary fall on *any* bit.
- **Subnet masks & CIDR**: a mask is a run of `1`s (network) followed by `0`s (host); `/n` simply counts the network bits.
- **Private address space (RFC 1918)**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` — reused everywhere behind **NAT**, never routed on the public internet.
- **Why subnet?** One network is **one broadcast domain** — a single Sales broadcast (ARP, DHCP Discover…) floods IT and Accounting too. Splitting one block into right-sized subnets gives each zone its own broadcast domain *and* conserves addresses.
- **How to subnet (four moves)**: ① count the need (how many subnets, and hosts in the largest) → ② pick the smallest prefix where `usable = 2^(host bits) − 2` covers it → ③ walk the blocks (subnets start at multiples of `256 / 2ⁿ`) → ④ name the edges (network = host bits all `0`, broadcast = all `1`, gateway = first usable).
- **IP design best practices**: plan by function (one subnet per role), size for growth (round up, leave gaps), and **document it** — an IPAM sheet of every subnet, mask, gateway, and purpose.

**Part 3 — Network Design Foundations**
- **Physical topologies**: **star** (every device to a central switch — today's standard), **bus** (one shared backbone — cheap, legacy), **ring** (token-passing loop), **mesh** (many cross-links — resilient, costly, used for backbones).
- **Frame delivery modes**: **unicast** (one recipient), **broadcast** (every host on the segment), and **multicast** (a subscribed group) — and how a switch (or a VLAN on it) joins its ports into the single broadcast domain a subnet maps to.


### Break (10 min)
- Step away, stretch, and grab a drink before diving into the hands-on labs.

### Hands-on Lab

> [!TIP]
> Each lab below is a short introduction. **Full step-by-step instructions, diagrams, and lab questions live in the companion guides** — follow those pages while you work.

---

**Lab A — Wireshark: TCP 3-Way Handshake**

Capture live traffic and watch a real **`SYN → SYN-ACK → ACK`** handshake set up a TCP connection. You'll filter with `tcp.flags.syn == 1`, expand the TCP **Flags** to confirm each step, use **Follow TCP Stream** to read the whole conversation, then contrast it with a connectionless **UDP** (DNS) exchange that has no handshake at all.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Wireshark — Lab A: The TCP 3-Way Handshake](./WIRESHARK_GUIDE.md)**

---

**Lab B — Cisco Packet Tracer: IP Network Design & Multi-Subnet Build**

> [!IMPORTANT]
> This lab builds the foundation for all future Packet Tracer exercises. Students design an IP scheme **from scratch**.

Design and build a real, routed network for a company with three departments — **Engineering** (50 hosts), **Marketing** (25 hosts), **Management** (10 hosts) — out of `192.168.0.0/24`. You'll subnet on paper first (`/26`, `/27`, `/28`), build the three-department topology, configure the router, verify same-subnet **and** cross-subnet connectivity, then deliberately break it (wrong mask, IP conflict) to learn why addressing discipline matters.

> 📖 **Full subnet tables, router config, verification, and questions:**
> 👉 **[Packet Tracer — Lab B: Multi-Subnet IP Design Build](./PACKET_TRACER_GUIDE.md#lab-b--multi-subnet-ip-design-build)**

---

### Deep Dive: IP Addressing & Subnetting

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

### Homework
- Kurose & Ross Wireshark Lab: **TCP** (download from textbook website)
- **IP Design Exercise**: Given the network `10.0.0.0/16`, design an IP scheme for 5 departments with 100, 60, 30, 15, and 5 hosts. Document subnet addresses, masks, usable ranges, and gateways in a table
- In Packet Tracer: Build the design above with at least 1 PC per subnet and verify cross-subnet connectivity via a router

---