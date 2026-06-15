## Session 4 — Routing, Switching & VLANs

> 🌐 **English** | [日本語](./README.ja.md)

- [Session 4 — Routing, Switching \& VLANs](#session-4--routing-switching--vlans)
  - [Lecture](#lecture)
  - [Break (10 min)](#break-10-min)
  - [Hands-on Lab](#hands-on-lab)
  - [Deep Dive: VLANs \& 802.1Q Trunking](#deep-dive-vlans--8021q-trunking)
    - [Access vs. Trunk Ports](#access-vs-trunk-ports)
    - [The 802.1Q Tag (4 bytes)](#the-8021q-tag-4-bytes)
    - [Inter-VLAN Routing: two ways](#inter-vlan-routing-two-ways)
  - [Deep Dive: Switching, Routing \& Useful `show` Commands](#deep-dive-switching-routing--useful-show-commands)
    - [MAC Table vs. Routing Table](#mac-table-vs-routing-table)
    - [Static vs. Dynamic Routing](#static-vs-dynamic-routing)
    - [Administrative Distance (who wins when sources disagree)](#administrative-distance-who-wins-when-sources-disagree)
    - [Verify Commands (Packet Tracer / IOS)](#verify-commands-packet-tracer--ios)
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


### Break (10 min)
- Step away, stretch, and grab a drink before diving into the hands-on labs.

### Hands-on Lab

> [!TIP]
> Each lab below is a short introduction. **Full step-by-step instructions, diagrams, and lab questions live in the companion guides** — follow those pages while you work.

---

**Lab A — Cisco Packet Tracer: "Small Office Network" Mega-Lab**

> [!IMPORTANT]
> This is the core infrastructure lab of the course. Students build a realistic office network from scratch.

Build a 6-PC, 2-switch, 1-router office with **three VLANs** (Admin / Sales / IT). You'll create the VLANs on both switches, assign **access ports**, **trunk** the switch-to-switch and switch-to-router links, configure **Router-on-a-Stick** sub-interfaces (`G0/0.10/.20/.30` with `encapsulation dot1Q`) as per-VLAN gateways, add **per-VLAN DHCP**, then prove inter-VLAN routing and verify with `show vlan brief` / `show interfaces trunk` / `show ip route`.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Packet Tracer — Lab A: Small Office Mega-Lab](./PACKET_TRACER_GUIDE.md#lab-a--small-office-network-mega-lab)**

---

**Lab B — Cisco Packet Tracer: Routing Comparison — Static vs. RIP vs. OSPF**

Build **one** 3-router topology (R1–R2–R3 in a line, each with a LAN behind it) and make PC1 ping PC3 **three different ways** — so you can feel the trade-offs between the routing methods first-hand on the *same* network.

1. **Static** — hand-type `ip route` on every router for each remote LAN. It works, but notice how many lines you wrote and that nothing recovers if a link drops.
2. **RIP** — wipe the static routes, enable `router rip` / `version 2` / `network …`. Watch routes appear automatically; check the **hop-count** metric and AD **120** in `show ip route`.
3. **OSPF** — remove RIP, enable `router ospf 1` + `network … area 0`. Compare its **cost** metric (bandwidth-based) and AD **110**, then unplug the R1–R2 link and watch OSPF **reconverge** onto the backup path.

Verify each stage with `show ip route`, `show ip protocols`, and `traceroute` from PC1 to PC3 — the routing table's source code (`S` / `R` / `O`) tells you which method is in charge.

> 📖 **Full instructions, figures, and questions:**
> 👉 **[Packet Tracer — Lab B: Routing Comparison (Static / RIP / OSPF)](./PACKET_TRACER_GUIDE.md#lab-b--routing-comparison-static-vs-rip-vs-ospf)**

---

### Deep Dive: VLANs & 802.1Q Trunking

#### Access vs. Trunk Ports

| | **Access port** | **Trunk port** |
|:---|:---|:---|
| **Carries** | exactly **one** VLAN | **many** VLANs |
| **Frames** | untagged | **802.1Q-tagged** (except the native VLAN) |
| **Connects to** | end devices (PC, printer, AP) | another switch, or a router (Router-on-a-Stick) |
| **Config** | `switchport mode access` + `switchport access vlan N` | `switchport mode trunk` |

#### The 802.1Q Tag (4 bytes)

Inserted into the Ethernet header right after the **source MAC**:

| Field | Bits | Meaning |
|:---|:---:|:---|
| **TPID** | 16 | always `0x8100` — the flag that says "an 802.1Q tag follows" |
| **PCP** | 3 | priority / QoS class (0–7) |
| **DEI** | 1 | drop-eligible indicator |
| **VLAN ID** | 12 | the VLAN number (**1–4094**) |

> [!NOTE]
> The **native VLAN** (default VLAN 1) is the one VLAN sent **untagged** on a trunk. Both ends must agree on it — a *native VLAN mismatch* causes traffic to leak between VLANs.

#### Inter-VLAN Routing: two ways

| Approach | How it works | Best when |
|:---|:---|:---|
| **Router-on-a-Stick (ROAS)** | one trunk link to a router with **one sub-interface per VLAN** (`g0/0.10`, `.20`…) tagged via `encapsulation dot1Q` | a few VLANs, using a router you already have |
| **Layer-3 switch (SVI)** | a `vlan N` **Switched Virtual Interface** holds the gateway IP and routes in hardware | many VLANs, high throughput |

---

### Deep Dive: Switching, Routing & Useful `show` Commands

#### MAC Table vs. Routing Table

| | **Switch** MAC address table | **Router** routing table |
|:---|:---|:---|
| **Layer** | 2 (MAC) | 3 (IP) |
| **Maps** | MAC → switch **port** | destination **network** → next hop / interface |
| **Learns by** | reading the **source MAC** of arriving frames | connected interfaces, static routes, routing protocols |
| **Unknown entry** | **floods** out every port | drops it (or uses the default route) |
| **Ages out** | ~5 min idle | static = never; dynamic = protocol timers |

#### Static vs. Dynamic Routing

| | **Static** | **Dynamic** (RIP / OSPF) |
|:---|:---|:---|
| **Set by** | typed by hand (`ip route …`) | routers advertise & learn automatically |
| **Scales to** | small networks | medium → large |
| **Reacts to failure** | no (manual fix) | yes (reconverges) |
| **Overhead** | none | CPU/bandwidth for routing updates |

#### Administrative Distance (who wins when sources disagree)

| Route source | AD |
|:---|:---:|
| Connected interface | **0** |
| Static route | **1** |
| OSPF | **110** |
| RIP | **120** |
| Unknown / unreachable | **255** |

> [!TIP]
> **Lower AD wins.** A static route (AD 1) beats an OSPF-learned route (AD 110) to the *same* network. The **metric** only breaks ties **within** a single protocol.

#### Verify Commands (Packet Tracer / IOS)

| Command | Shows |
|:---|:---|
| `show vlan brief` | which ports are in which VLAN |
| `show interfaces trunk` | trunk links, allowed VLANs, native VLAN |
| `show mac address-table` | learned MAC → port mappings |
| `show ip route` | the routing table (connected / static / dynamic) |
| `show ip interface brief` | interfaces, IPs, up/down status |
| `show ip arp` | the router's IP → MAC cache |

> [!NOTE]
> **Packet walk-through (one frame, PC → PC across a router):** the **Layer-3 IP** addresses stay the same end-to-end, but the **Layer-2 MAC** addresses are **rewritten at every hop**, and the IP **TTL drops by 1** at each router — the three facts that tie this whole session together.

---

### Homework
- Save the Small Office `.pkt` file for future practice
- Research: What is OSPF and how does it differ from static routing? Write a 1-paragraph summary
- Optional: Try configuring port security on a switch port — set max MAC addresses to 1

---