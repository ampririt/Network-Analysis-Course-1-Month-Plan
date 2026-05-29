## Session 4 — Routing, Switching & VLANs

- [Session 4 — Routing, Switching \& VLANs](#session-4--routing-switching--vlans)
  - [📖 Lecture](#-lecture)
  - [Break (10 min)](#break-10-min)
  - [🛠️ Hands-on Lab](#️-hands-on-lab)
  - [📚 Homework](#-homework)


**Learning Objectives**: Understand how switches forward traffic using MAC address tables. Configure VLANs to segment networks. Learn how routers forward traffic between subnets. Analyze routing & switching in captures.

### 📖 Lecture 
- **Switching**: MAC address table, broadcast vs. unicast, switch forwarding decisions
- **VLANs**: Why segment networks? VLAN tagging (802.1Q), trunk vs. access ports
- **Routing**: Routing table, static vs. dynamic routing, default gateway
- **Inter-VLAN Routing**: Router-on-a-Stick concept, sub-interfaces
- **Subnetting Refresher**: Network/host portions, subnet mask, CIDR notation
- Live demo: `show ip route`, `show vlan brief`, `show mac address-table` in Packet Tracer

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