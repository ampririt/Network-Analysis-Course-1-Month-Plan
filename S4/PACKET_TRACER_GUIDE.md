## Cisco Packet Tracer — Lab A: Small Office Network (VLANs & Routing)

> 🌐 **English** | [日本語](./PACKET_TRACER_GUIDE.ja.md)

> Companion guide to [Session 4 — Routing, Switching & VLANs](./README.md).
> Read this **before** doing **Lab A** or **Lab B** in the [README](./README.md#hands-on-lab).
> New to Packet Tracer? Start with the [Session 1 Packet Tracer guide](../S1/PACKET_TRACER_GUIDE.md) for installation, the window tour, and CLI basics. This session uses the VLAN/Router-on-a-Stick concepts that Sessions 2–3 deliberately deferred.

---

- [Cisco Packet Tracer — Lab A: Small Office Network (VLANs \& Routing)](#cisco-packet-tracer--lab-a-small-office-network-vlans--routing)
- [Lab A — "Small Office Network" Mega-Lab](#lab-a--small-office-network-mega-lab)
  - [Addressing \& VLAN plan](#addressing--vlan-plan)
  - [Step A1 — Build the topology](#step-a1--build-the-topology)
  - [Step A2 — Create the VLANs (both switches)](#step-a2--create-the-vlans-both-switches)
  - [Step A3 — Assign access ports](#step-a3--assign-access-ports)
  - [Step A4 — Trunk the switch-to-switch and switch-to-router links](#step-a4--trunk-the-switch-to-switch-and-switch-to-router-links)
  - [Step A5 — Router-on-a-Stick (inter-VLAN routing)](#step-a5--router-on-a-stick-inter-vlan-routing)
  - [Step A6 — DHCP per VLAN](#step-a6--dhcp-per-vlan)
  - [Step A7 — Test \& verify](#step-a7--test--verify)
  - [Lab A Questions](#lab-a-questions)
- [Exercise Practice](#exercise-practice)
  - [Exercise 1 — Add a 4th VLAN (Guest)](#exercise-1--add-a-4th-vlan-guest)
  - [Exercise 2 — Static route to a "server" network](#exercise-2--static-route-to-a-server-network)
  - [Exercise 3 — Port security (1 MAC per access port)](#exercise-3--port-security-1-mac-per-access-port)
  - [Exercise 4 — ACL: isolate the Guest VLAN](#exercise-4--acl-isolate-the-guest-vlan)
  - [Exercise 5 — Break it \& fix it (troubleshooting drill)](#exercise-5--break-it--fix-it-troubleshooting-drill)
- [Lab B — Routing Comparison (Static vs. RIP vs. OSPF)](#lab-b--routing-comparison-static-vs-rip-vs-ospf)
  - [Addressing plan](#addressing-plan)
  - [Step B1 — Build the topology](#step-b1--build-the-topology)
  - [Step B2 — Base addressing (do this once)](#step-b2--base-addressing-do-this-once)
  - [Stage 1 — Static routing](#stage-1--static-routing)
  - [Stage 2 — RIP (dynamic, distance-vector)](#stage-2--rip-dynamic-distance-vector)
  - [Stage 3 — OSPF (dynamic, link-state) + reconvergence](#stage-3--ospf-dynamic-link-state--reconvergence)
  - [Static vs. RIP vs. OSPF — side by side](#static-vs-rip-vs-ospf--side-by-side)
  - [Lab B Questions](#lab-b-questions)
  - [Lab B — Exercise Practice](#lab-b--exercise-practice)
    - [Exercise 1 — Default route instead of two static routes](#exercise-1--default-route-instead-of-two-static-routes)
    - [Exercise 2 — Watch the AD tie-break (static vs. OSPF on one router)](#exercise-2--watch-the-ad-tie-break-static-vs-ospf-on-one-router)
    - [Exercise 3 — Make OSPF *prefer* the path through R2](#exercise-3--make-ospf-prefer-the-path-through-r2)
    - [Exercise 4 — Add a loopback "server" and advertise it](#exercise-4--add-a-loopback-server-and-advertise-it)
    - [Exercise 5 — Time the reconvergence (RIP vs. OSPF)](#exercise-5--time-the-reconvergence-rip-vs-ospf)
- [Next Steps](#next-steps)

---

## Lab A — "Small Office Network" Mega-Lab

**Objective:** build a realistic office network from scratch — three VLANs, a trunk, **Router-on-a-Stick** inter-VLAN routing, and per-VLAN DHCP.

> [!IMPORTANT]
> This is the **core infrastructure lab** of the course. Everything here — VLANs, trunking, sub-interfaces — is what real switches and routers do every day.

### Addressing & VLAN plan

| VLAN | Name | Subnet | Gateway (router sub-int) | PCs |
|:---:|:---|:---|:---|:---|
| 10 | Admin | `192.168.10.0/24` | `192.168.10.1` | 2 |
| 20 | Sales | `192.168.20.0/24` | `192.168.20.1` | 2 |
| 30 | IT | `192.168.30.0/24` | `192.168.30.1` | 2 |

**Topology:** 6 PCs → 1 Switch → 1 Router. The reference build below uses **one** `2960` switch (all three VLANs live on it). In a real office you would split the PCs across **two** switches joined by a trunk — that variant is covered as an optional extension in [Step A4](#step-a4--trunk-the-switch-to-switch-and-switch-to-router-links).

### Step A1 — Build the topology

Drag on **6 PCs**, **1 switch** (`2960`), and **1 router** (`2911`). Cable each PC to a switch access port and connect the switch to the router — use **Copper Straight-Through** for every link (PC↔switch and switch↔router are unlike devices, so straight-through is correct; you'd only need a crossover for switch↔switch on very old gear).

Group the PCs by department so the topology mirrors the addressing plan: **PC0/PC1 → Admin (VLAN 10)**, **PC2/PC3 → Sales (VLAN 20)**, **PC4/PC5 → IT (VLAN 30)**.

> 💡 New links show an **amber dot** at each end while the port initialises, then turn **green** when the link is up. The switch↔router link may stay amber/red until you `no shutdown` the router interface in Step A5 — that's expected.

<p align="center">
  <img src="./img/labA-step1-topology.png" width="820" alt="Lab A single-switch office topology"><br>
  <em>Fig. 1 — The office topology: six PCs grouped into Admin/Sales/IT, all homed on one <code>2960-24TT</code> switch, with a <code>2911</code> router on top providing inter-VLAN routing.</em>
</p>

To configure a device, click it and open the **CLI** tab. On a brand-new switch/router, press **Enter** to skip the initial setup wizard (answer **no** if it asks about the configuration dialog), then type `enable` to reach privileged EXEC mode (`Switch>` → `Switch#`).

<p align="center">
  <img src="./img/switch_CLI.png" width="780" alt="Opening the switch CLI tab in Packet Tracer"><br>
  <em>Fig. 1b — Click the switch → <strong>CLI</strong> tab to reach the IOS command line. Everything in Steps A2–A4 is typed here.</em>
</p>

### Step A2 — Create the VLANs (both switches)

A **VLAN** (Virtual LAN) splits one physical switch into several independent Layer-2 networks — each VLAN is its own broadcast domain. Creating a VLAN adds it to the switch's **VLAN database**; the `name` is just a human label (Admin/Sales/IT) — it's the **number** (10/20/30) that the switch and router actually use to tag traffic.

Type these in the switch CLI (if you build the two-switch variant, repeat the exact same block on **each** switch so both know all three VLANs):

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
  <img src="./img/labA-step2-vlans.png" width="720" alt="Creating VLANs 10, 20 and 30 in the switch CLI"><br>
  <em>Fig. 2 — Creating the three VLANs. Each <code>vlan &lt;id&gt;</code> drops you into <code>config-vlan</code> mode where you set its <code>name</code>; <code>exit</code> returns to global config.</em>
</p>

Confirm they exist with `show vlan brief`. At this point the three new VLANs are **active** but have **no ports yet** — every access port still sits in the default **VLAN 1** until you assign it in Step A3.

<p align="center">
  <img src="./img/show_vlan_brief.png" width="720" alt="show vlan brief listing VLANs 10, 20, 30 active"><br>
  <em>Fig. 2b — <code>show vlan brief</code>: VLANs 10/20/30 are <strong>active</strong>. Notice all <code>Fa</code> ports are still under VLAN 1 — we move them next.</em>
</p>

### Step A3 — Assign access ports

An **access port** carries exactly **one** VLAN — the port a PC plugs into. Frames on an access port travel **untagged**; the switch internally remembers which VLAN the port belongs to. `interface range` lets you configure several ports in one go, and `switchport mode access` pins the port as an end-device port (so it never tries to negotiate a trunk). Put each PC's port in the right VLAN to match the addressing plan:

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
  <img src="./img/access_port_setting.png" width="720" alt="Assigning fa0/1-6 to VLANs 10, 20 and 30 in access mode"><br>
  <em>Fig. 3 — Access-port assignment: <code>fa0/1-2</code>→VLAN 10, <code>fa0/3-4</code>→VLAN 20, <code>fa0/5-6</code>→VLAN 30. Re-run <code>show vlan brief</code> and you'll now see those ports listed under their VLANs instead of VLAN 1.</em>
</p>

### Step A4 — Trunk the switch-to-switch and switch-to-router links

A **trunk port** carries **all** VLANs at once, tagging each frame with its VLAN ID using **802.1Q**. In this single-switch build, the one link that must be a trunk is the **switch → router** uplink (`Gig0/1`) — that's the cable the router-on-a-stick will ride on:

```ios
Switch(config)# interface gig0/1
Switch(config-if)# switchport mode trunk
```

> 💡 An **access port** = one VLAN (to end devices). A **trunk port** = many VLANs (switch→router here, and switch→switch in larger designs). The trunk is what lets one cable carry Admin, Sales, and IT traffic simultaneously — each frame stamped with its 802.1Q tag so the far end knows which VLAN it belongs to.

> **Two-switch extension:** if you split the PCs across two switches, the cable **between the switches** must *also* be a trunk — run the same `switchport mode trunk` on the inter-switch port on **both** sides, so VLANs 10/20/30 can cross from one switch to the other.

<p align="center">
  <img src="./img/trunk_port_setting.png" width="640" alt="Setting Gig0/1 to trunk mode"><br>
  <em>Fig. 4 — Turning the switch→router uplink <code>Gig0/1</code> into a trunk.</em>
</p>

Verify with `show interfaces trunk`. `Gig0/1` should be **trunking** with encapsulation **802.1q**, and VLANs **1,10,20,30** listed as *allowed and active*. The **native VLAN** is 1 (untagged frames fall back to VLAN 1) — fine for the lab.

<p align="center">
  <img src="./img/show_interfaces_trunk.png" width="720" alt="show interfaces trunk output showing Gig0/1 trunking VLANs 1,10,20,30"><br>
  <em>Fig. 4b — <code>show interfaces trunk</code> confirms <code>Gig0/1</code> is trunking and actively carrying VLANs 10, 20 and 30 (plus native VLAN 1).</em>
</p>

### Step A5 — Router-on-a-Stick (inter-VLAN routing)

VLANs are separate networks, so they **can't talk without a router**. We give the router **one sub-interface per VLAN** on a single physical link — **Router-on-a-Stick (RoaS)**. The physical interface (`g0/0`) carries no IP itself; we just bring it up with `no shutdown`. Each **sub-interface** (`g0/0.10`, `g0/0.20`, `g0/0.30`) tags with `encapsulation dot1Q <vlan>` and holds that VLAN's **gateway IP** — the address each PC will use as its default gateway.

> ### 📡 How routing works in this lab
>
> **Routing = choosing which interface to send a packet out of, based on its destination IP.** Two layers are doing two different jobs:
>
> - **The switch is Layer 2.** It only moves frames *within* a single VLAN, by MAC address. It has no way to get a packet from `192.168.10.x` to `192.168.20.x`.
> - **The router is Layer 3.** Moving a packet *between* subnets (VLANs) is its job. Without it, the three VLANs are three isolated islands.
>
> **Where the router gets its "map":** because each sub-interface owns an IP *inside* a VLAN's subnet, the router automatically learns all three networks as **connected (`C`) routes** — no manual `ip route` needed. You can see them later with `show ip route` (Fig. 7).
>
> **The path of one cross-VLAN ping (Admin PC0 → Sales PC2):**
> 1. PC0 sees the destination is a different subnet → sends the packet to its **default gateway** `192.168.10.1` (the router's `g0/0.10`). *(The first ping often times out here while PC0 ARPs for the gateway's MAC.)*
> 2. The frame crosses the **trunk** `Gig0/1` tagged as **VLAN 10**.
> 3. The router strips the tag, **routes** on destination `192.168.20.3` → connected route → out `g0/0.20`.
> 4. It re-tags the packet as **VLAN 20** and sends it back down the trunk; the switch delivers it to PC2.
>
> **The proof — `TTL=127`:** a PC sends with TTL 128, and **every router hop subtracts 1**. The replies in Step A7 show 127, which is hard evidence the packet was **routed exactly one hop** through the router. Same-VLAN traffic (switched, not routed) would still read 128.

Open the **router** CLI the same way (Fig. 1b), skipping the setup dialog:

<p align="center">
  <img src="./img/router_cli.png" width="700" alt="Router CLI: declining the setup dialog and entering config mode"><br>
  <em>Fig. 5a — Answer <code>no</code> to the initial configuration dialog, press RETURN, then <code>enable</code> → <code>configure terminal</code>.</em>
</p>

Now build the sub-interfaces:

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

> **`encapsulation dot1Q 10`** tells the sub-interface "frames here belong to VLAN 10 — read/write the 802.1Q tag accordingly." That's how one physical wire serves three gateways. The sub-interface number (`.10`) is just a label by convention; it's the `encapsulation dot1Q` value that actually binds it to the VLAN.

<p align="center">
  <img src="./img/Router-on-a-Stick.png" width="760" alt="Router sub-interface configuration for VLANs 10, 20, 30"><br>
  <em>Fig. 5 — The three sub-interfaces being created: each gets its own <code>encapsulation dot1Q</code> tag and gateway IP, all on the single physical <code>g0/0</code>.</em>
</p>

Verify with `show ip interface brief`: the physical `GigabitEthernet0/0` is **up/up** (but unassigned), and `g0/0.10/.20/.30` each carry their gateway IP and read **up/up**.

<p align="center">
  <img src="./img/show_ip_interface_brief.png" width="760" alt="show ip interface brief showing sub-interfaces up with gateway IPs"><br>
  <em>Fig. 5b — <code>show ip interface brief</code>: each sub-interface is up with its <code>.1</code> gateway address. If a sub-interface shows <strong>down/down</strong>, you forgot <code>no shutdown</code> on the physical <code>g0/0</code>.</em>
</p>

### Step A6 — DHCP per VLAN

Give each VLAN its own DHCP pool so PCs auto-address instead of you typing IPs by hand. It's the same `ip dhcp pool` pattern, one per VLAN: `network` defines the subnet to hand out, and `default-router` is the gateway the clients receive — which is exactly the sub-interface IP you set in Step A5. (Optionally add `dns-server` for a DNS address, and `ip dhcp excluded-address` to reserve IPs you don't want leased.)

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

<p align="center">
  <img src="./img/DHCP_per_VLAN.png" width="640" alt="Configuring three ip dhcp pools, one per VLAN"><br>
  <em>Fig. 6 — Three DHCP pools on the router — VLAN10/20/30 — each handing out its subnet with the matching gateway as <code>default-router</code>.</em>
</p>

Then set each PC to **DHCP**: click the PC → **Config** tab → **FastEthernet0** (or **Desktop → IP Configuration**) → select **DHCP**. The PC broadcasts a request, the router answers from the right pool, and the address + gateway appear automatically.

<p align="center">
  <img src="./img/PC_DHCP_setting.png" width="700" alt="PC0 set to DHCP, receiving gateway 192.168.10.1"><br>
  <em>Fig. 6b — PC0 switched to <strong>DHCP</strong>: it picks up an address in <code>192.168.10.0/24</code> with default gateway <code>192.168.10.1</code> — proof the VLAN 10 pool and gateway are wired correctly.</em>
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
  <img src="./img/show_ip_route.png" width="760" alt="show ip route listing the three connected VLAN subnets"><br>
  <em>Fig. 7 — <code>show ip route</code>: the router has a <strong>C</strong> (connected) and <strong>L</strong> (local) route for each VLAN subnet via its sub-interface. These connected routes are what let it forward between VLANs.</em>
</p>

Now the real test — ping **across** VLANs. From an **Admin** PC (VLAN 10) ping a **Sales** PC (VLAN 20):

<p align="center">
  <img src="./img/ping_PC0-%3EPC2.png" width="560" alt="Ping from Admin PC0 to Sales PC2 succeeding"><br>
  <em>Fig. 7b — Admin <code>→</code> Sales ping. The <strong>first</strong> request often times out while the PC ARPs for its gateway; the rest reply with <strong>0% loss</strong>. <code>TTL=127</code> (not 128) shows the packet was <strong>routed one hop</strong> — the router decremented the TTL, proving inter-VLAN routing worked.</em>
</p>

<p align="center">
  <img src="./img/ping_PC2-%3EPC0.png" width="560" alt="Ping from Sales PC2 back to Admin PC0 succeeding"><br>
  <em>Fig. 7c — The reverse path, Sales <code>→</code> Admin, also at 0% loss — traffic flows both ways.</em>
</p>

<p align="center">
  <img src="./img/ping_test.png" width="820" alt="Topology view of the ping path between PC0 and PC2"><br>
  <em>Fig. 7d — The same ping seen on the topology: the two PCs sit in different VLANs/subnets, yet reach each other through the router-on-a-stick.</em>
</p>

> 💡 **Why the first ping drops:** the source PC has no MAC for its gateway yet, so it sends an ARP request and the first ICMP echo expires before the reply arrives. This is normal — only a *persistent* failure means something is misconfigured (check the [Q4 troubleshooting answer](#lab-a-questions)).

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

## Exercise Practice

First make sure the base lab is finished — these are the things your Small Office Network should already do:

- [ ] 3 VLANs created on the switch; PC ports in the right VLANs (`show vlan brief`)
- [ ] the switch→router uplink is a **trunk** carrying VLANs 10/20/30 (`show interfaces trunk`)
- [ ] Router-on-a-Stick sub-interfaces up; **inter-VLAN ping succeeds**
- [ ] each VLAN gets DHCP addresses automatically

Now extend it. Each exercise builds on the lab you just made — **try it, then check your result with the listed `show`/ping command.**

### Exercise 1 — Add a 4th VLAN (Guest)

Add **VLAN 40 "Guest", `192.168.40.0/24`, gateway `192.168.40.1`** end-to-end: create the VLAN, put `fa0/7` in it (access mode), add the router sub-interface `g0/0.40` with `encapsulation dot1Q 40`, and a DHCP pool `VLAN40`. Plug in a new PC set to DHCP.

> ✅ **Verify:** the new PC gets a `192.168.40.x` address and can ping an Admin PC. `show ip route` now lists a 4th connected subnet.
>
> 💡 *Hint:* it's the **same four steps** (A2 → A3 → A5 → A6) you already did, just with the number 40.

### Exercise 2 — Static route to a "server" network

Add a second router (or a loopback) representing a server subnet `10.0.0.0/24` and give your main router a **static route** to it:

```ios
Router(config)# ip route 10.0.0.0 255.255.255.0 <next-hop-ip>
```

> ✅ **Verify:** `show ip route` shows an **`S`** (static) entry for `10.0.0.0/24`, and a VLAN PC can ping into that network.
>
> 💡 *Hint:* a static route says "to reach *this* destination, send packets to *that* next hop." Compare it to the automatic **`C`** connected routes the VLANs already have.

### Exercise 3 — Port security (1 MAC per access port)

Lock each Admin access port to a single MAC address so a rogue device can't take over the port:

```ios
Switch(config)# interface range fa0/1-2
Switch(config-if-range)# switchport port-security
Switch(config-if-range)# switchport port-security maximum 1
Switch(config-if-range)# switchport port-security violation shutdown
```

> ✅ **Verify:** `show port-security interface fa0/1`. Swap the PC for a different one (or change its MAC) and confirm the port goes **err-disabled**.

### Exercise 4 — ACL: isolate the Guest VLAN

Write a standard/extended ACL so the **Guest VLAN (40)** can reach the internet/server network but **cannot** reach Admin (10), then apply it to `g0/0.40`.

> ✅ **Verify:** a Guest PC can still ping the server subnet but its ping to an Admin PC now **fails** ("Destination host unreachable"), while Admin↔Sales still works.
>
> 💡 *Hint:* `access-list` to define the rule, `ip access-group <n> in` on the sub-interface to apply it. Remember the implicit `deny any` at the end.

### Exercise 5 — Break it & fix it (troubleshooting drill)

Have a classmate (or your future self) introduce **one** fault, then diagnose it with `show` commands only:

| Sabotage | Symptom you'll see | The `show` that finds it |
|:---|:---|:---|
| Put `fa0/1` in the wrong VLAN | PC can't reach its gateway | `show vlan brief` |
| `shutdown` the router's `g0/0` | *all* inter-VLAN ping fails | `show ip interface brief` |
| Change a sub-interface `encapsulation` number | one VLAN can't route | `show running-config` |
| Set the uplink back to `access` mode | only 1 VLAN works through the trunk | `show interfaces trunk` |

> ✅ **Goal:** find and fix the fault **without** looking at what was changed — this is exactly how real network troubleshooting works.



## Lab B — Routing Comparison (Static vs. RIP vs. OSPF)

**Objective:** build **one** small 3-router network, then make `PC1` reach `PC3` **three different ways** — hand-typed **static** routes, then **RIP**, then **OSPF** — on the *same* topology. Same wires, same IPs; only the routing method changes, so you can feel each one's trade-offs directly.

> [!IMPORTANT]
> The whole point is the **comparison**. Don't tear the lab down between stages — you'll *remove* the previous method and *add* the next one on the same routers, then re-run `show ip route` and `traceroute` to see what changed.

> [!TIP]
> The routing table's left-hand **code letter** is the scoreboard: **`C`** connected, **`S`** static, **`R`** RIP, **`O`** OSPF. When you switch methods, watch the codes for the two remote LANs flip `S` → `R` → `O`.

### Addressing plan

Three routers in a **triangle** (every router links to the other two — that redundancy is what lets us demo OSPF reconvergence later). `PC1` lives behind `R1`, `PC3` behind `R3`; `R2` is a transit router.

| Device | Interface | IP / mask | Connects to |
|:---|:---|:---|:---|
| **R1** | `Gig0/0` | `192.168.1.1 /24` | PC1 LAN |
| | `Gig0/1` | `10.0.12.1 /30` | R2 |
| | `Gig0/2` | `10.0.13.1 /30` | R3 |
| **R2** | `Gig0/1` | `10.0.12.2 /30` | R1 |
| | `Gig0/2` | `10.0.23.1 /30` | R3 |
| **R3** | `Gig0/0` | `192.168.3.1 /24` | PC3 LAN |
| | `Gig0/1` | `10.0.23.2 /30` | R2 |
| | `Gig0/2` | `10.0.13.2 /30` | R1 |
| **PC1** | `Fa0` | `192.168.1.10 /24`, GW `192.168.1.1` | R1 |
| **PC3** | `Fa0` | `192.168.3.10 /24`, GW `192.168.3.1` | R3 |

> The `/30` links use `255.255.255.252` (two usable hosts — perfect for a point-to-point router link). The two LANs we care about reaching are **`192.168.1.0/24`** (PC1) and **`192.168.3.0/24`** (PC3).

### Step B1 — Build the topology

Drag on **3 routers** (`2911`) and **2 PCs**. The `2911` has three Gigabit ports (`Gig0/0–0/2`) — exactly enough for the triangle plus a LAN. Cable with **Copper Straight-Through**:

- PC1 ↔ R1 `Gig0/0`, PC3 ↔ R3 `Gig0/0`
- R1 `Gig0/1` ↔ R2 `Gig0/1` (the R1–R2 link)
- R2 `Gig0/2` ↔ R3 `Gig0/1` (the R2–R3 link)
- R1 `Gig0/2` ↔ R3 `Gig0/2` (the direct R1–R3 link — this is the backup path)

> 💡 Router interfaces are **administratively down by default**. Every link will sit **red** until you `no shutdown` the interface in Step B2 — that's expected, not a cabling fault.

<p align="center">
  <img src="./img/routing/topology.png" width="820" alt="Lab B triangle topology: PC1–R1, PC3–R3, and R1/R2/R3 in a triangle"><br>
  <em>Fig. B1 — The Lab B topology. <code>PC1</code> (<code>192.168.1.10/24</code>) sits behind <code>R1</code>; <code>PC3</code> (<code>192.168.3.10/24</code>) behind <code>R3</code>; <code>R2</code> is a transit router. The three routers form a <strong>triangle</strong>, so there are <strong>two</strong> ways from R1 to R3 — the direct <code>10.0.13.0/30</code> link and the longer path through R2. That redundancy is exactly what lets us demo OSPF reconvergence in Stage 3. Every link shows <strong>red</strong> here because the interfaces are still shut down.</em>
</p>

📸 *Capture your topology and each stage's `show ip route` / `traceroute` output into [`S4/img/`](./img/) for your report.*

### Step B2 — Base addressing (do this once)

This part is **identical for all three stages** — it just brings the interfaces up with their IPs. Routing comes *after*. Open each router's **CLI** tab and type its block.

**R1:**
```ios
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface g0/1
R1(config-if)# ip address 10.0.12.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface g0/2
R1(config-if)# ip address 10.0.13.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# end
```

**R2:**
```ios
Router(config)# hostname R2
R2(config)# interface g0/1
R2(config-if)# ip address 10.0.12.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface g0/2
R2(config-if)# ip address 10.0.23.1 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# end
```

**R3:**
```ios
Router(config)# hostname R3
R3(config)# interface g0/0
R3(config-if)# ip address 192.168.3.1 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# interface g0/1
R3(config-if)# ip address 10.0.23.2 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# interface g0/2
R3(config-if)# ip address 10.0.13.2 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# end
```

Set **PC1** (`192.168.1.10`, mask `255.255.255.0`, gateway `192.168.1.1`) and **PC3** (`192.168.3.10` / `255.255.255.0` / `192.168.3.1`) statically via **Desktop → IP Configuration**.

> ✅ **Checkpoint:** `show ip interface brief` on each router shows every used interface **up/up**. Run `show ip route` — you'll see only **`C`/`L`** (connected/local) routes for that router's *own* links. **PC1 → PC3 ping still fails** — no router knows how to reach the *other* end's LAN yet. That's the problem the next three stages each solve a different way.

<p align="center">
  <img src="./img/routing/R1_ip_set.png" width="760" alt="R1 show ip interface brief — g0/0, g0/1, g0/2 all up/up"><br>
  <img src="./img/routing/R2_ip_set.png" width="760" alt="R2 show ip interface brief — g0/1, g0/2 up/up, g0/0 unused"><br>
  <img src="./img/routing/R3_ip_set.png" width="760" alt="R3 show ip interface brief — g0/0, g0/1, g0/2 all up/up"><br>
  <em>Fig. B2 — <code>show ip interface brief</code> on R1, R2 and R3 after Step B2. Every interface that carries a cable reads <strong>up / up</strong> with its planned address (R2 only uses two links, so its <code>g0/0</code> stays <em>administratively down</em>). The always-present <code>Vlan1</code> is unused here — ignore it. <strong>Status</strong> = Layer 1 (is the cable/line up); <strong>Protocol</strong> = Layer 2 (is the line usable). Both must say <em>up</em> before routing can work.</em>
</p>

<p align="center">
  <img src="./img/routing/PC1%20%E2%86%92%20PC3%20ping%20still%20fails.png" width="560" alt="PC1 ping to 192.168.3.10 fails with Destination host unreachable"><br>
  <em>Fig. B2b — The checkpoint failure, and it's the <em>expected</em> result. <code>ping 192.168.3.10</code> from PC1 returns <strong>"Reply from 192.168.1.1: Destination host unreachable"</strong> — note the reply comes from <strong>R1's own gateway IP</strong>, not from PC3. R1 received the packet but has <strong>no route</strong> to the <code>192.168.3.0/24</code> LAN, so it drops it and tells PC1 so. Each of the next three stages fixes this a different way.</em>
</p>

### Stage 1 — Static routing

You teach each router every remote network **by hand** with `ip route <dest> <mask> <next-hop>`. To get PC1 ↔ PC3 working we route each LAN toward the other end. Use the **direct R1–R3 link** as the path:

**R1** — reach PC3's LAN via R3:
```ios
R1(config)# ip route 192.168.3.0 255.255.255.0 10.0.13.2
```

**R3** — reach PC1's LAN via R1:
```ios
R3(config)# ip route 192.168.1.0 255.255.255.0 10.0.13.1
```

> ✅ **Verify:** `show ip route` on R1 now lists an **`S`** entry `192.168.3.0/24 [1/0] via 10.0.13.2` — note **AD 1**. `ping`/`traceroute` from **PC1 to 192.168.3.10** succeeds; the trace shows `10.0.13.2` (R3) then PC3 — the direct R1→R3 link.

<p align="center">
  <img src="./img/routing/ip_route_R1R3.png" width="760" alt="R3 show ip route with a static S route to 192.168.1.0/24 via 10.0.13.1"><br>
  <em>Fig. B3 — The static route as seen on <strong>R3</strong>: <code>S&nbsp;&nbsp;192.168.1.0/24 [1/0] via 10.0.13.1</code>. The <strong>S</strong> code = hand-typed static; <code>[1/0]</code> = <strong>administrative distance 1</strong> (static is the most-trusted source) and metric 0. Everything else is still <strong>C</strong>/<strong>L</strong> — only the one remote LAN was added by hand. R1 carries the mirror-image <code>S&nbsp;192.168.3.0/24 via 10.0.13.2</code>.</em>
</p>

<p align="center">
  <img src="./img/routing/PC1%20to%20192.168.3.10%20succeeds.png" width="520" alt="PC1 ping to 192.168.3.10 succeeds, 0% loss, TTL=126">
  <img src="./img/routing/PC1%20to%20192.168.3.10%20succeeds%20%28tracert%29.png" width="520" alt="PC1 tracert to 192.168.3.10 showing 192.168.1.1, 10.0.13.2, then 192.168.3.10"><br>
  <em>Fig. B3b — <strong>Left:</strong> the second <code>ping</code> reaches PC3 at <strong>0% loss</strong> (the first attempt loses two packets while ARP resolves the next-hops). <strong>TTL=126</strong> is the evidence: PC1 sends at 128 and <strong>two routers</strong> (R1 then R3) each decrement it by one. <strong>Right:</strong> <code>tracert</code> spells the path out — hop 1 <code>192.168.1.1</code> (R1), hop 2 <code>10.0.13.2</code> (R3 on the direct link), hop 3 <code>192.168.3.10</code> (PC3).</em>
</p>

> 💡 **Feel the cost of static:** that was only *two* networks and it already took two hand-typed lines pointing at exact next-hops. Add PC2's LAN, or a 4th router, and you're editing **every** router by hand — and if the `10.0.13.0` link dies, nothing fails over. That pain is the entire reason dynamic routing exists.

**Before Stage 2, remove the static routes** so they don't mask what RIP learns:
```ios
R1(config)# no ip route 192.168.3.0 255.255.255.0 10.0.13.2
R3(config)# no ip route 192.168.1.0 255.255.255.0 10.0.13.1
```

### Stage 2 — RIP (dynamic, distance-vector)

Now the routers **advertise their networks to each other** and learn the rest automatically. RIP's metric is **hop count** (max 15; 16 = unreachable), and its administrative distance is **120**.

> ### RIPv1 vs RIPv2 — and why we force `version 2`
>
> RIP comes in two versions, and the difference matters in this lab:
>
> | | **RIPv1** (1988, RFC 1058) | **RIPv2** (RFC 2453) |
> |:---|:---|:---|
> | **Subnet mask in updates** | ❌ **No** — classful only | ✅ **Yes** — carries the prefix (VLSM/CIDR) |
> | **How updates are sent** | **Broadcast** `255.255.255.255` | **Multicast** `224.0.0.9` (quieter) |
> | **Authentication** | none | optional (plain-text / MD5) |
> | **Status** | legacy / obsolete | the version you actually use |
>
> The killer difference is the **subnet mask**. Our links are a mix of prefixes — `/24` LANs and `/30` router links — all carved out of networks that don't sit on classful boundaries. **RIPv1 can't carry a mask**, so a receiving router just *guesses* the classful default (`10.0.0.0` → `/8`, `192.168.x.0` → `/24`) and our `/30` links would break. **RIPv2 sends the real mask with every route**, so VLSM works. That's why every router gets an explicit `version 2`.
>
> Two paired commands make RIPv2 behave:
> - **`version 2`** — send/receive v2 updates only (by default a Cisco router *sends* v1 and *listens* for both, a recipe for silent mismatches).
> - **`no auto-summary`** — stops RIP from auto-summarising back to the classful boundary at network edges. Without it, the three `10.0.x.x /30` links would collapse into one `10.0.0.0/8` advertisement and the routers couldn't tell them apart.

Enable RIPv2 on all three routers and list each router's *directly connected* classful networks:

**R1:**
```ios
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# no auto-summary
R1(config-router)# network 192.168.1.0
R1(config-router)# network 10.0.0.0
```

**R2:**
```ios
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# no auto-summary
R2(config-router)# network 10.0.0.0
```

**R3:**
```ios
R3(config)# router rip
R3(config-router)# version 2
R3(config-router)# no auto-summary
R3(config-router)# network 192.168.3.0
R3(config-router)# network 10.0.0.0
```

> 💡 The RIP `network` command takes a **classful** address, so `network 10.0.0.0` covers *all three* `10.0.x.x /30` links in one line. `no auto-summary` keeps the `/30` and `/24` prefixes intact across the classful `10.0.0.0` boundary.

> ✅ **Verify:** give it ~30 s, then `show ip route` on R1 shows `192.168.3.0/24` as an **`R`** route — `[120/1]` = **AD 120, 1 hop** (it learned the direct R1→R3 path; via R2 would be 2 hops and loses). `show ip protocols` confirms RIP is running and lists the advertised networks. PC1 → PC3 works again — but this time **nobody typed a route to it**.

<p align="center">
  <img src="./img/routing/RIP_ip_route_table.png" width="780" alt="R1 show ip route with R (RIP) routes [120/1], plus the timestamp counting up"><br>
  <em>Fig. B4 — R1's table under RIP. The two remote networks now carry the <strong>R</strong> code: <code>R&nbsp;192.168.3.0/24 [120/1] via 10.0.13.2</code> and <code>R&nbsp;10.0.23.0/30 [120/1]</code> learned over <strong>both</strong> uplinks (RIP load-balances equal-cost paths). <code>[120/1]</code> = <strong>AD 120, 1 hop</strong>. The <code>00:00:23</code> field is the time since the last update — RIP refreshes every <strong>30 s</strong>, so it counts up to 30 then resets; if it ever passes 180 s (the invalid timer) the route is on its way out. Compare to Fig. B3: same destinations, but learned automatically instead of typed by hand.</em>
</p>

**Before Stage 3, turn RIP off:**
```ios
R1(config)# no router rip
R2(config)# no router rip
R3(config)# no router rip
```

### Stage 3 — OSPF (dynamic, link-state) + reconvergence

OSPF builds a full **map** of the topology and picks paths by **cost** (derived from link bandwidth), with administrative distance **110**. Networks are matched with a **wildcard mask** and placed in an **area** (we use `area 0`, the backbone). Enable on all three:

**R1:**
```ios
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
R1(config-router)# network 10.0.13.0 0.0.0.3 area 0
```

**R2:**
```ios
R2(config)# router ospf 1
R2(config-router)# network 10.0.12.0 0.0.0.3 area 0
R2(config-router)# network 10.0.23.0 0.0.0.3 area 0
```

**R3:**
```ios
R3(config)# router ospf 1
R3(config-router)# network 192.168.3.0 0.0.0.255 area 0
R3(config-router)# network 10.0.23.0 0.0.0.3 area 0
R3(config-router)# network 10.0.13.0 0.0.0.3 area 0
```

> 💡 The **wildcard mask** is the inverse of the subnet mask: `/30` → `0.0.0.3`, `/24` → `0.0.0.255`. It tells OSPF *which interfaces* fall in each `network` statement. Watch the console — you'll see an **adjacency** message (`FULL`) as neighbours discover each other.

> ✅ **Verify:** `show ip route` on R1 shows `192.168.3.0/24` as an **`O`** route with `[110/2]` — **AD 110**, and a **cost** of 2 (two GigE links on the direct R1→R3 path: the link plus R3's LAN). `show ip ospf neighbor` lists the adjacencies in state **FULL**. PC1 → PC3 still works.

<p align="center">
  <img src="./img/routing/ospf_ip_route_table.png" width="780" alt="R1 show ip route with O (OSPF) routes [110/2] to the remote networks"><br>
  <em>Fig. B5 — The same R1 table, now under OSPF. The remote networks flip to the <strong>O</strong> code: <code>O&nbsp;192.168.3.0/24 [110/2] via 10.0.13.2</code>. <code>[110/2]</code> = <strong>AD 110</strong> (lower than RIP's 120, so if both ran OSPF would win) and a <strong>cost of 2</strong> — OSPF's metric is bandwidth-derived, not hop count. Watch the codes march across the three stages for the very same destination: <strong>S → R → O</strong>. Only the <em>method</em> changed; the connected <strong>C</strong>/<strong>L</strong> routes never moved.</em>
</p>

**The payoff — watch OSPF reconverge.** The direct R1↔R3 link is the active path. Now **break it** and watch OSPF heal automatically:

```ios
R1(config)# interface g0/2
R1(config-if)# shutdown
```

> ✅ **Verify:** within a few seconds `show ip route` on R1 re-installs `192.168.3.0/24` via **R2** (`[110/3]` — now 3 hops worth of cost through `10.0.12.2`). `traceroute` from PC1 to PC3 now shows the **extra hop through R2** that wasn't there before. **No one touched a routing command** — OSPF recomputed the map and rerouted on its own. Bring the link back with `no shutdown` and it reverts to the direct path. *Static routing would simply have gone dark here.*

<p align="center">
  <img src="./img/routing/ip_route_inf_g0%3A2_shutdown_case.png" width="860" alt="R1 shutting g0/2: OSPF adjacency drops to DOWN and the route to 192.168.3.0/24 re-installs as [110/3] via 10.0.12.2"><br>
  <em>Fig. B6 — The moment of reconvergence. Shutting <code>g0/2</code> prints <code>%OSPF-5-ADJCHG: ... Nbr 192.168.3.1 ... from FULL to DOWN</code> — R1 instantly notices the neighbour is gone. The follow-up <code>do show ip route</code> shows the cost has climbed to <code>O&nbsp;192.168.3.0/24 [110/<strong>3</strong>] via 10.0.12.2</code>: the path now runs R1→R2→R3, and the dead <code>10.0.13.0/30</code> link has dropped out of the table entirely.</em>
</p>

<p align="center">
  <img src="./img/routing/The_payoff.png" width="820" alt="Topology after the break: the direct R1–R3 link is red/down while the R1–R2 and R2–R3 links stay green"><br>
  <em>Fig. B6b — The healed topology. The direct R1↔R3 link is <strong>red (down)</strong>, but every other link is <strong>green</strong> and PC1 still reaches PC3 — traffic simply detours through R2. This self-healing is what <strong>reconvergence</strong> means, and it cost zero configuration.</em>
</p>

<p align="center">
  <img src="./img/routing/comparing_before%3Aafter_break%20%28tracert%29.png" width="600" alt="Two tracerts: before the break 3 hops via 10.0.13.2; after the break 4 hops via 10.0.12.2 and 10.0.23.2"><br>
  <em>Fig. B6c — The proof from PC1, before and after, in one shot. <strong>Top</strong> (direct link up): 3 hops — <code>192.168.1.1 → 10.0.13.2 → 192.168.3.10</code>. <strong>Bottom</strong> (direct link down): 4 hops — <code>192.168.1.1 → 10.0.12.2 → 10.0.23.2 → 192.168.3.10</code>, the new detour through R2 made visible. The extra line *is* the reroute.</em>
</p>

### Static vs. RIP vs. OSPF — side by side

| | **Static** | **RIP** | **OSPF** |
|:---|:---|:---|:---|
| **Code in `show ip route`** | `S` | `R` | `O` |
| **Administrative distance** | **1** | **120** | **110** |
| **Metric** | none (you pick the next-hop) | **hop count** (max 15) | **cost** (bandwidth-based) |
| **How routes are set** | typed by hand per router | routers advertise & learn | routers flood a link-state map |
| **Reacts to a link failure** | ❌ no — manual fix | ✅ yes (slow, ~minutes) | ✅ yes (fast, seconds) |
| **Config effort / scaling** | grows with every network | low | low (more setup, scales best) |
| **Good for** | tiny / stub networks | small networks | medium → large networks |

> [!NOTE]
> All three put the **same** `C`/`L` connected routes on each router — those never change. What differs is how the **remote** LANs (`192.168.1.0` and `192.168.3.0`) get learned, and what happens when a link drops. That's the whole lesson in one table.

### Lab B Questions

**Try each one first, then click "Show answer".**

**Q1.** After enabling RIP, the route to `192.168.3.0/24` shows `[120/1]`. What do the **120** and the **1** mean?

<details>
<summary>💡 Show answer</summary>

**120** is RIP's **administrative distance** (how trustworthy the *source* is), and **1** is the **metric** — here the **hop count**, one router away. AD is compared *between* protocols; metric breaks ties *within* one protocol.
</details>

**Q2.** You enable **both** static and OSPF routes to `192.168.3.0/24` at the same time. Which one does the router install, and why?

<details>
<summary>💡 Show answer</summary>

The **static** route. Administrative distance decides when sources disagree — **static = 1** beats **OSPF = 110**, and **lower AD wins**. The OSPF route stays in the database but isn't installed in the routing table.
</details>

**Q3.** When you `shutdown` the direct R1–R3 link under OSPF, PC1 still reaches PC3. Under **static** routing the same break would fail. Why the difference?

<details>
<summary>💡 Show answer</summary>

OSPF keeps a **live map** of the whole topology, so when a link goes down every router **recomputes** and installs the next-best path (via R2) automatically — **reconvergence**. A static route is a fixed, hand-typed instruction with no awareness of link state, so it just keeps pointing at a dead next-hop until a human fixes it.
</details>

**Q4.** Why is the OSPF `network` statement written `network 10.0.12.0 0.0.0.3 area 0` instead of `network 10.0.12.0 255.255.255.252`?

<details>
<summary>💡 Show answer</summary>

OSPF uses a **wildcard mask** (the inverse of the subnet mask), not the subnet mask itself. `/30` = `255.255.255.252`, whose inverse is `0.0.0.3`. The wildcard tells OSPF exactly which interface addresses fall inside this `network` statement, and `area 0` places them in the backbone area.
</details>

---

### Lab B — Exercise Practice

First confirm the base lab works — by the end of Stage 3 your 3-router network should already do all of this:

- [ ] every used interface is **up/up** (`show ip interface brief` on R1/R2/R3)
- [ ] **PC1 → PC3 ping succeeds** under each method in turn (static, then RIP, then OSPF)
- [ ] the remote-LAN code in `show ip route` flips **`S` → `R` → `O`** as you switch methods
- [ ] breaking the direct R1–R3 link under **OSPF** reroutes through R2 (`traceroute` gains a hop); under **static** it goes dark
- [ ] you removed the previous method (`no ip route…` / `no router rip`) before adding the next, so nothing is masked

Now extend it. Each exercise builds on the same triangle — **try it, then check with the listed `show`/`traceroute` command.** Capture each result into [`S4/img/`](./img/) for your report.

#### Exercise 1 — Default route instead of two static routes

On R1, replace the specific static route to PC3's LAN with a **default route** `0.0.0.0/0`, and do the mirror on R3. Useful when a router has only one way out (a stub).

```ios
R1(config)# no ip route 192.168.3.0 255.255.255.0 10.0.13.2
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.13.2
```

> ✅ **Verify:** `show ip route` shows `S*  0.0.0.0/0 [1/0] via 10.0.13.2` and **`Gateway of last resort is 10.0.13.2`** at the top. PC1 → PC3 still pings. The `*` marks it as the **candidate default**.

#### Exercise 2 — Watch the AD tie-break (static vs. OSPF on one router)

With **OSPF still running**, add a static route to the *same* destination on R1 and prove which one the router installs:

```ios
R1(config)# ip route 192.168.3.0 255.255.255.0 10.0.13.2
```

> ✅ **Verify:** `show ip route 192.168.3.0` now shows the **`S`** route, not `O` — **static AD 1 beats OSPF AD 110**, lower wins. Remove it (`no ip route 192.168.3.0 255.255.255.0 10.0.13.2`) and within seconds the **`O`** route returns. This is the [Lab B Q2](#lab-b-questions) answer, seen live.

#### Exercise 3 — Make OSPF *prefer* the path through R2

Cost follows bandwidth, so you can steer OSPF without shutting anything down. Raise the cost of R1's direct link to R3 so the R1→R2→R3 path wins **while both links are up**:

```ios
R1(config)# interface g0/2
R1(config-if)# ip ospf cost 100
```

> ✅ **Verify:** `show ip route` re-installs `192.168.3.0/24` via `10.0.12.2` (R2), and `traceroute` from PC1 shows the R2 hop — even though the direct link is still up. Set it back with `no ip ospf cost`. *This is traffic engineering: same topology, chosen path.*

#### Exercise 4 — Add a loopback "server" and advertise it

Give R3 a loopback that stands in for a server, and let the routing protocol carry it end-to-end (no static route):

```ios
R3(config)# interface loopback 0
R3(config-if)# ip address 8.8.8.8 255.255.255.255
R3(config-if)# exit
R3(config)# router ospf 1
R3(config-router)# network 8.8.8.8 0.0.0.0 area 0
```

> ✅ **Verify:** `show ip route` on **R1** lists `O  8.8.8.8/32` it never had configured locally, and PC1 can ping `8.8.8.8`. The `/32` wildcard `0.0.0.0` advertises exactly that one host address.

#### Exercise 5 — Time the reconvergence (RIP vs. OSPF)

Re-run the "break the direct link" test under **each** protocol and time how long PC1 loses PC3. Start a continuous ping (`ping 192.168.3.10 -t` on the PC, or repeated pings) and shut `R1 g0/2`.

> ✅ **Verify:** under **OSPF** connectivity returns in **a few seconds**; under **RIP** it can take **tens of seconds to minutes** (hold-down + update timers). This is the practical meaning of the "Reacts to a link failure" row in the [comparison table](#static-vs-rip-vs-ospf--side-by-side).



## Next Steps

- Optional — analyse traffic in [Wireshark](./WIRESHARK_GUIDE.md): capture/open 802.1Q-tagged frames and find the VLAN tag and per-hop TTL decrements from Lab A.
- **Homework (from the README):** save the Small Office `.pkt`, write a paragraph on **OSPF vs static routing**, and try **port security** (max 1 MAC per access port).
- The Small Office network is a great base for later experiments — add a DMZ, an ACL restricting a VLAN, or a second switch link for redundancy (STP).
