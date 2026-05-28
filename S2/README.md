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