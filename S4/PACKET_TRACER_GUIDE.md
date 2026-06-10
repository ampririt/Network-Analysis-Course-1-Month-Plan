## Cisco Packet Tracer — Lab A: Small Office Network (VLANs & Routing)

> Companion guide to [Session 4 — Routing, Switching & VLANs](./README.md).
> Read this **before** doing **Lab A** in the [README](./README.md#hands-on-lab).
> New to Packet Tracer? Start with the [Session 1 Packet Tracer guide](../S1/PACKET_TRACER_GUIDE.md) for installation, the window tour, and CLI basics. This session uses the VLAN/Router-on-a-Stick concepts that Sessions 2–3 deliberately deferred.

---

- [Cisco Packet Tracer — Lab A: Small Office Network (VLANs & Routing)](#cisco-packet-tracer--lab-a-small-office-network-vlans--routing)
  - [Lab A — "Small Office Network" Mega-Lab](#lab-a--small-office-network-mega-lab)
    - [Addressing & VLAN plan](#addressing--vlan-plan)
    - [Step A1 — Build the topology](#step-a1--build-the-topology)
    - [Step A2 — Create the VLANs (both switches)](#step-a2--create-the-vlans-both-switches)
    - [Step A3 — Assign access ports](#step-a3--assign-access-ports)
    - [Step A4 — Trunk the switch-to-switch and switch-to-router links](#step-a4--trunk-the-switch-to-switch-and-switch-to-router-links)
    - [Step A5 — Router-on-a-Stick (inter-VLAN routing)](#step-a5--router-on-a-stick-inter-vlan-routing)
    - [Step A6 — DHCP per VLAN](#step-a6--dhcp-per-vlan)
    - [Step A7 — Test & verify](#step-a7--test--verify)
    - [Lab A Questions](#lab-a-questions)
  - [Self-Check](#self-check)
  - [Next Steps](#next-steps)

---

## Lab A — "Small Office Network" Mega-Lab

**⏱️ ~55 min · Objective:** build a realistic office network from scratch — three VLANs, a trunk, **Router-on-a-Stick** inter-VLAN routing, and per-VLAN DHCP.

> [!IMPORTANT]
> This is the **core infrastructure lab** of the course. Everything here — VLANs, trunking, sub-interfaces — is what real switches and routers do every day.

### Addressing & VLAN plan

| VLAN | Name | Subnet | Gateway (router sub-int) | PCs |
|:---:|:---|:---|:---|:---|
| 10 | Admin | `192.168.10.0/24` | `192.168.10.1` | 2 |
| 20 | Sales | `192.168.20.0/24` | `192.168.20.1` | 2 |
| 30 | IT | `192.168.30.0/24` | `192.168.30.1` | 2 |

**Topology:** 6 PCs → 2 Switches → 1 Router (+ 1 Server optional).

### Step A1 — Build the topology

Drag on **6 PCs**, **2 switches** (`2960`), **1 router** (`2911`). Cable each PC to a switch access port, link the two switches together, and connect one switch to the router (Copper Straight-Through throughout).

<p align="center">
  <!-- ![Lab A topology](./img/labA-step1-topology.png) -->
  <em>Fig. 1 — 📸 <code>img/labA-step1-topology.png</code>: the 6-PC, 2-switch, 1-router office topology.</em>
</p>

### Step A2 — Create the VLANs (both switches)

Do this on **each** switch so both know all three VLANs:

```ios
Switch> enable
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name Admin
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name Sales
Switch(config-vlan)# exit
Switch(config)# vlan 30
Switch(config-vlan)# name IT
Switch(config-vlan)# end
```

<p align="center">
  <!-- ![Create VLANs](./img/labA-step2-vlans.png) -->
  <em>Fig. 2 — 📸 <code>img/labA-step2-vlans.png</code>: the three VLANs created (verify with <code>show vlan brief</code>).</em>
</p>

### Step A3 — Assign access ports

An **access port** carries exactly **one** VLAN — the port a PC plugs into. Put each PC's port in the right VLAN:

```ios
Switch(config)# interface range fa0/1-2
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# exit
Switch(config)# interface range fa0/3-4
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# exit
Switch(config)# interface range fa0/5-6
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 30
```

<p align="center">
  <!-- ![Access ports](./img/labA-step3-accessports.png) -->
  <em>Fig. 3 — 📸 <code>img/labA-step3-accessports.png</code>: PC ports assigned to their VLANs in access mode.</em>
</p>

### Step A4 — Trunk the switch-to-switch and switch-to-router links

A **trunk port** carries **all** VLANs at once, tagging each frame with its VLAN ID (802.1Q). The link **between the two switches** and the link **from the switch to the router** must both be trunks:

```ios
Switch(config)# interface gig0/1
Switch(config-if)# switchport mode trunk
```

> 💡 An **access port** = one VLAN (to end devices). A **trunk port** = many VLANs (between switches, and switch→router). The trunk is what lets one cable carry Admin, Sales, and IT traffic simultaneously.

<p align="center">
  <!-- ![Trunk ports](./img/labA-step4-trunk.png) -->
  <em>Fig. 4 — 📸 <code>img/labA-step4-trunk.png</code>: trunk links carrying all VLANs (verify with <code>show interfaces trunk</code>).</em>
</p>

### Step A5 — Router-on-a-Stick (inter-VLAN routing)

VLANs are separate networks, so they **can't talk without a router**. We give the router **one sub-interface per VLAN** on a single physical link — **Router-on-a-Stick**. Each sub-interface tags with `encapsulation dot1Q` and holds that VLAN's **gateway IP**:

```ios
Router> enable
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit
Router(config)# interface g0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit
Router(config)# interface g0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.30.1 255.255.255.0
Router(config-subif)# end
```

> **`encapsulation dot1Q 10`** tells the sub-interface "frames here belong to VLAN 10 — read/write the 802.1Q tag accordingly." That's how one physical wire serves three gateways.

<p align="center">
  <!-- ![Router-on-a-Stick](./img/labA-step5-roas.png) -->
  <em>Fig. 5 — 📸 <code>img/labA-step5-roas.png</code>: the router sub-interfaces, one gateway per VLAN.</em>
</p>

### Step A6 — DHCP per VLAN

Give each VLAN its own DHCP pool so PCs auto-address — the same `ip dhcp pool` pattern, one per VLAN.

```ios
Router(config)# ip dhcp pool VLAN10
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.10.1
Router(dhcp-config)# exit
Router(config)# ip dhcp pool VLAN20
Router(dhcp-config)# network 192.168.20.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.20.1
Router(dhcp-config)# exit
Router(config)# ip dhcp pool VLAN30
Router(dhcp-config)# network 192.168.30.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.30.1
Router(dhcp-config)# end
```

Then set each PC to **DHCP** (Desktop → IP Configuration → DHCP).

<p align="center">
  <!-- ![DHCP per VLAN](./img/labA-step6-dhcp.png) -->
  <em>Fig. 6 — 📸 <code>img/labA-step6-dhcp.png</code>: PCs receiving addresses from their VLAN's pool.</em>
</p>

### Step A7 — Test & verify

1. **Inter-VLAN ping:** from an **Admin** PC, `ping` a **Sales** PC → should succeed *via the router*.
2. Verify the build with these `show` commands on the switch/router:

   | Command | What it confirms |
   |:---|:---|
   | `show vlan brief` | which ports are in which VLAN |
   | `show interfaces trunk` | which links are trunks + allowed VLANs |
   | `show ip interface brief` | sub-interfaces up with their gateway IPs |
   | `show ip route` | the three connected VLAN subnets |

<p align="center">
  <!-- ![Verify](./img/labA-step7-verify.png) -->
  <em>Fig. 7 — 📸 <code>img/labA-step7-verify.png</code>: a successful cross-VLAN ping + verification output.</em>
</p>

### Lab A Questions

**Try each one first, then click "Show answer".**

**Q1.** What's the difference between an **access port** and a **trunk port**?

<details>
<summary>💡 Show answer</summary>

An **access port** belongs to **one VLAN** and connects to an end device (a PC) — frames leave it **untagged**. A **trunk port** carries **many VLANs** between switches (and switch→router), **tagging** each frame with its 802.1Q VLAN ID so the other end knows which VLAN it belongs to.
</details>

**Q2.** Why can't an Admin PC reach a Sales PC **without the router**, even though they're on the same switches?

<details>
<summary>💡 Show answer</summary>

VLANs are **separate broadcast domains = separate IP subnets**. Layer-2 switching only moves frames *within* a VLAN. To cross from VLAN 10's subnet to VLAN 20's subnet you need **Layer-3 routing** — that's what the router's sub-interfaces (the per-VLAN gateways) provide. No router, no inter-VLAN traffic.
</details>

**Q3.** What does **`encapsulation dot1Q 20`** do on sub-interface `g0/0.20`?

<details>
<summary>💡 Show answer</summary>

It binds that sub-interface to **VLAN 20** and tells it to **add/read the 802.1Q tag** for VLAN 20 on the trunk. This is what lets a **single physical interface** act as the gateway for multiple VLANs at once (Router-on-a-Stick) — each sub-interface handles one VLAN's tagged traffic.
</details>

**Q4.** In `show vlan brief`, a PC can't reach its gateway. What's the first thing to check?

<details>
<summary>💡 Show answer</summary>

Confirm the PC's **switch port is in the correct VLAN** (and in `access` mode). A very common mistake is the port defaulting to VLAN 1 or being left as a trunk. Also check the **trunk** to the router is up and **allows that VLAN**, and that the router sub-interface for the VLAN is configured and `no shutdown`.
</details>

---

## Self-Check

- [ ] **Lab A:** 3 VLANs created on both switches; PC ports in the right VLANs (`show vlan brief`)
- [ ] **Lab A:** switch–switch and switch–router links are **trunks** (`show interfaces trunk`)
- [ ] **Lab A:** Router-on-a-Stick sub-interfaces up; **inter-VLAN ping succeeds**
- [ ] **Lab A:** each VLAN gets DHCP addresses automatically
- [ ] Screenshots saved into [`S4/img/`](./img/) and rendering in this guide

---

## Next Steps

- Analyse your own VLAN traffic in [Wireshark Lab B](./WIRESHARK_GUIDE.md): capture/open 802.1Q-tagged frames and find the VLAN tag and per-hop TTL decrements you just engineered.
- **Homework (from the README):** save the Small Office `.pkt`, write a paragraph on **OSPF vs static routing**, and try **port security** (max 1 MAC per access port).
- The Small Office network is a great base for later experiments — add a DMZ, an ACL restricting a VLAN, or a second switch link for redundancy (STP).
