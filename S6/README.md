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