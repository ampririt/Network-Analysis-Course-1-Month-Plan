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