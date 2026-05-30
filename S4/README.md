## Session 4 — Routing, Switching & VLANs

- [Session 4 — Routing, Switching \& VLANs](#session-4--routing-switching--vlans)
  - [📖 Lecture](#-lecture)
  - [Break (10 min)](#break-10-min)
  - [🛠️ Hands-on Lab](#️-hands-on-lab)
  - [📚 Homework](#-homework)


**Learning Objectives**: Understand how switches forward traffic using MAC address tables. Configure VLANs to segment networks. Learn how routers forward traffic between subnets. Analyze routing & switching in captures.

### 📖 Lecture 
- **Layer 2 vs. Layer 3**: Where switching ends and routing begins; collision domains vs. broadcast domains
- **Switching**: MAC address table, MAC learning & aging, broadcast vs. unicast vs. multicast, switch forwarding decisions, flooding of unknown unicast frames
- **VLANs**: Why segment networks? (security, broadcast control, logical grouping), VLAN tagging (802.1Q), trunk vs. access ports, the native VLAN
- **Spanning Tree Protocol (STP)**: Loop prevention, root bridge election, port states (blocking/forwarding) — why a redundant switch link won't melt the network
- **Routing**: Routing table, static vs. dynamic routing, default gateway, longest-prefix match, administrative distance & metrics
- **Dynamic Routing Protocols**: Distance-vector (RIP) vs. link-state (OSPF) at a high level — how routers learn paths automatically
- **Inter-VLAN Routing**: Router-on-a-Stick concept, sub-interfaces, vs. Layer 3 switch (SVI) approach
- **Subnetting Refresher**: Network/host portions, subnet mask, CIDR notation, calculating usable hosts & subnet ranges
- **First-Hop & Address Services**: Default gateway role, ARP at Layer 2, DHCP relay across subnets
- **Packet Walk-through**: Trace a single packet PC → switch → router → switch → PC, watching MAC rewrites vs. IP preservation at each hop
- Live demo: `show ip route`, `show vlan brief`, `show mac address-table`, `show interfaces trunk`, `show ip arp` in Packet Tracer

### Break (10 min)
- Step away, stretch, and grab a drink before diving into the hands-on labs.

### 🛠️ Hands-on Lab

**Lab A — Cisco Packet Tracer: "Small Office Network" Mega-Lab (55 min)**

> [!IMPORTANT]
> This is the core infrastructure lab of the course. Students build a realistic office network from scratch.

**Topology**: 6 PCs → 2 Switches → 1 Router → 1 Server

1. **Create 3 VLANs** on both switches:
   - VLAN 10: Admin (2 PCs)
   - VLAN 20: Sales (2 PCs)
   - VLAN 30: IT (2 PCs)
2. **Assign switch ports** to VLANs (access mode)
3. **Configure trunk port** between switches
4. **Configure Router-on-a-Stick**:
   - Sub-interfaces: `G0/0.10`, `G0/0.20`, `G0/0.30`
   - Assign gateway IPs for each VLAN
   - Enable `encapsulation dot1Q [vlan-id]`
5. **Configure DHCP** on router for each VLAN
6. **Test inter-VLAN connectivity**: Ping from Admin PC to Sales PC
7. **Verify** with: `show vlan brief`, `show ip interface brief`, `show ip route`

**Lab B — Wireshark: VLAN & Routing Traffic Analysis (25 min)**
1. Open a provided PCAP containing 802.1Q tagged traffic
2. Filter: `vlan` → identify VLAN IDs in captured frames
3. Inspect the Ethernet II header → locate the 802.1Q tag field (TPID `0x8100`)
4. Filter: `icmp` → trace a ping across subnets — observe TTL decrement at each router hop
5. Use **Statistics → Endpoints** to identify all active subnets in the capture
6. Use **Statistics → Conversations** to map traffic flow between VLANs

### 📚 Homework
- Save the Small Office `.pkt` file for future practice
- Research: What is OSPF and how does it differ from static routing? Write a 1-paragraph summary
- Optional: Try configuring port security on a switch port — set max MAC addresses to 1

---