## Session 4 — Routing, Switching & VLANs

- [Session 4 — Routing, Switching \& VLANs](#session-4--routing-switching--vlans)
  - [Lecture](#lecture)
  - [Break (10 min)](#break-10-min)
  - [Hands-on Lab](#hands-on-lab)
    - [Packet Tracer — Lab A + Design Challenge guide →](./PACKET_TRACER_GUIDE.md)
    - [Wireshark — Lab B guide →](./WIRESHARK_GUIDE.md)
  - [Homework](#homework)


**Learning Objectives**: Understand how switches forward traffic using MAC address tables. Configure VLANs to segment networks. Learn how routers forward traffic between subnets. Analyze routing & switching in captures.

### Lecture 
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

### Hands-on Lab

> [!TIP]
> Each lab below is a short introduction. **Full step-by-step instructions, diagrams, and lab questions live in the companion guides** — follow those pages while you work.

---

**Lab A — Cisco Packet Tracer: "Small Office Network" Mega-Lab (~55 min)**

> [!IMPORTANT]
> This is the core infrastructure lab of the course. Students build a realistic office network from scratch.

Build a 6-PC, 2-switch, 1-router office with **three VLANs** (Admin / Sales / IT). You'll create the VLANs on both switches, assign **access ports**, **trunk** the switch-to-switch and switch-to-router links, configure **Router-on-a-Stick** sub-interfaces (`G0/0.10/.20/.30` with `encapsulation dot1Q`) as per-VLAN gateways, add **per-VLAN DHCP**, then prove inter-VLAN routing and verify with `show vlan brief` / `show interfaces trunk` / `show ip route`.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Packet Tracer — Lab A: Small Office Mega-Lab](./PACKET_TRACER_GUIDE.md#lab-a--small-office-network-mega-lab)**

---

**👥 Workgroup Design Challenge — Design It, Build It, Prove It (~40 min)**

> [!IMPORTANT]
> The **proof-of-competency** activity: one design challenge that shows students can turn requirements into a working network using everything from Sessions 2–4.

Given only a **business brief** (the "BrightByte" startup, base network `172.16.0.0/24`, three growth-sized departments + a server), each workgroup **designs an IP/VLAN scheme from scratch (VLSM), implements it in Packet Tracer, and demonstrates it passing a fixed 6-point acceptance test**. No step-by-step snippets — the students supply the design and apply Lab A's skills; success = the test passes.

> 📖 **The brief, deliverables, and acceptance test:**
> 👉 **[Workgroup Design Challenge](./PACKET_TRACER_GUIDE.md#workgroup-design-challenge--design-it-build-it-prove-it)**

---

**Lab B — Wireshark: VLAN & Routing Traffic Analysis (~25 min)**

Read VLANs and routing straight out of a capture. You'll filter `vlan` to find VLAN IDs, expand the Ethernet header to inspect the **802.1Q tag** (TPID `0x8100`), filter `icmp` to watch **TTL decrement at each router hop**, and use **Statistics → Endpoints / Conversations** to map the active subnets and inter-VLAN flows.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Wireshark — Lab B: VLAN & Routing Traffic Analysis](./WIRESHARK_GUIDE.md)**

### Homework
- Save the Small Office `.pkt` file for future practice
- Research: What is OSPF and how does it differ from static routing? Write a 1-paragraph summary
- Optional: Try configuring port security on a switch port — set max MAC addresses to 1

---