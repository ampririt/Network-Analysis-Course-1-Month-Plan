# 🛠️ Session 2 — Hands-on Lab Walkthrough

> **Companion to:** [Session 2 README](./README.md)
> **Goal:** Follow each lab step-by-step. Every step has a 📸 placeholder where you can drop a screenshot from your own machine into the [`img/`](./img/) folder.
>
> **How to add a picture:** save your screenshot into `S2/img/` (e.g. `S2/img/labA-step3.png`) and the matching `![...](./img/labA-step3.png)` line below will render it automatically.

---

## Contents
- [Before You Start](#before-you-start)
- [Lab A — Wireshark: TCP 3-Way Handshake](#lab-a--wireshark-tcp-3-way-handshake)
- [Lab B — Packet Tracer: Simulation-Mode Protocol Walk](#lab-b--packet-tracer-simulation-mode-protocol-walk)
- [Lab C — Packet Tracer: Multi-Subnet IP Design Build](#lab-c--packet-tracer-multi-subnet-ip-design-build)
- [Self-Check & Submission](#self-check--submission)
- [Is This Enough Practice for S2?](#is-this-enough-practice-for-s2)

---

## Before You Start

**Checklist:**
- [ ] Wireshark installed ([install guide](https://www.wireshark.org/docs/wsug_html_chunked/ChBuildInstallWinInstall.html))
- [ ] Cisco Packet Tracer installed ([install guide](https://www.netacad.com/skillsforall/files/Cisco_Packet_Tracer_Download_and_Installation_Instructions.pdf))
- [ ] Paper / spreadsheet ready for your subnet plan (Lab C)
- [ ] You have reviewed the header structures in the [README Lecture section](./README.md#-lecture)

> [!TIP]
> Keep the [Subnetting Quick-Reference Table](./README.md#subnetting-quick-reference-table) open in a second window during Lab C.

---

## Lab A — Wireshark: TCP 3-Way Handshake
**⏱️ ~20 min · Objective:** See a real `SYN → SYN-ACK → ACK` handshake and contrast TCP with UDP.

### Step A1 — Start a capture
1. Open Wireshark.
2. Double-click your active interface (the one with a moving traffic graph — usually `Wi-Fi` or `Ethernet`).
3. Capture begins immediately.

📸 *Picture: the interface list with the live traffic sparkline.*
<!-- ![Wireshark interface selection](./img/labA-step1-interfaces.png) -->

### Step A2 — Generate traffic
1. Leave Wireshark running.
2. In a browser, visit a **plain HTTP** site if possible (e.g. `http://example.com`) so the handshake is easy to spot.
3. Return to Wireshark and click the red **Stop** button after the page loads.

📸 *Picture: browser showing the loaded page + Wireshark still capturing.*
<!-- ![Generating traffic](./img/labA-step2-traffic.png) -->

### Step A3 — Filter for the handshake
1. In the filter bar type:
   ```
   tcp.flags.syn == 1
   ```
2. Press **Enter**. Now only packets with the SYN flag set are shown.

📸 *Picture: filter bar with the expression applied and rows highlighted.*
<!-- ![SYN filter applied](./img/labA-step3-synfilter.png) -->

### Step A4 — Identify SYN → SYN-ACK → ACK
Look for three consecutive packets between your PC and the server:

| # | Direction | Flags | Meaning |
|---|-----------|-------|---------|
| 1 | You → Server | `SYN` | "Can we talk?" |
| 2 | Server → You | `SYN, ACK` | "Yes — and you?" |
| 3 | You → Server | `ACK` | "Confirmed." |

Click each packet and expand **Transmission Control Protocol → Flags** in the detail pane to confirm.

📸 *Picture: the three handshake packets selected, Flags subtree expanded.*
<!-- ![3-way handshake](./img/labA-step4-handshake.png) -->

### Step A5 — Follow the TCP stream
1. Right-click any packet in the connection → **Follow → TCP Stream**.
2. Observe the full request/response conversation (red = your request, blue = server reply).

📸 *Picture: the Follow TCP Stream window.*
<!-- ![Follow TCP stream](./img/labA-step5-stream.png) -->

### Step A6 — Compare with UDP (DNS)
1. Close the stream window, clear the filter, and apply:
   ```
   udp.port == 53
   ```
2. Notice: **no handshake** — just query → response. This is connectionless UDP.

📸 *Picture: DNS query/response pair under the udp.port == 53 filter.*
<!-- ![DNS over UDP](./img/labA-step6-dns.png) -->

> [!NOTE]
> **What to write down:** the source/destination ports of your TCP connection, and why DNS doesn't need a handshake.

---

## Lab B — Packet Tracer: Simulation-Mode Protocol Walk
**⏱️ ~20 min · Objective:** Watch encapsulation (ICMP → IP → Ethernet) and ARP hop-by-hop.

### Step B1 — Build the topology
Drag onto the workspace and cable them as shown:

```
PC1 ──┐
      ├── Switch ── Router ── Server
PC2 ──┘
```
- PCs → Switch: **Copper Straight-Through**
- Switch → Router: **Copper Straight-Through**
- Router → Server: **Copper Straight-Through** (or place Server on a second switch)

📸 *Picture: the finished topology on the workspace.*
<!-- ![Lab B topology](./img/labB-step1-topology.png) -->

### Step B2 — Address the devices
Put PCs and the Server on **different subnets** so traffic must cross the router.

| Device | IP | Mask | Gateway |
|--------|------|------|---------|
| PC1 | `192.168.1.10` | `255.255.255.0` | `192.168.1.1` |
| PC2 | `192.168.1.11` | `255.255.255.0` | `192.168.1.1` |
| Server | `192.168.2.10` | `255.255.255.0` | `192.168.2.1` |

Router interfaces (Config tab or CLI):
```
Router> enable
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit
Router(config)# interface g0/1
Router(config-if)# ip address 192.168.2.1 255.255.255.0
Router(config-if)# no shutdown
```

📸 *Picture: PC IP Configuration window + router CLI showing interfaces up.*
<!-- ![Addressing](./img/labB-step2-addressing.png) -->

### Step B3 — Enter Simulation Mode
1. Click **Simulation** (bottom-right corner).
2. In **Edit Filters**, keep only **ICMP** and **ARP** checked to reduce noise.

📸 *Picture: Simulation panel with ICMP + ARP filters selected.*
<!-- ![Simulation mode](./img/labB-step3-simulation.png) -->

### Step B4 — Send a ping and step through
1. Use the **Add Simple PDU** (envelope) tool: click PC1, then the Server.
2. Click **Play / Capture-Forward** repeatedly to advance one hop at a time.
3. Watch the sequence:
   - **ARP request/reply** — PC1 learns the router's MAC (and the router learns the Server's).
   - **ICMP echo** travelling inside IP inside Ethernet.

📸 *Picture: a packet mid-flight between switch and router.*
<!-- ![Ping stepping](./img/labB-step4-ping.png) -->

### Step B5 — Inspect the PDU layers
1. Click a coloured packet square → the **PDU Information** window opens.
2. Use the **OSI Model** tab to read each layer; use **Inbound/Outbound PDU Details** to see real header fields (Ethernet → IP → ICMP).
3. Confirm the **destination MAC changes at the router** but the **destination IP stays the same** — the core idea of routing.

📸 *Picture: PDU Details showing the Ethernet + IP + ICMP headers.*
<!-- ![PDU inspection](./img/labB-step5-pdu.png) -->

> [!NOTE]
> **What to write down:** at how many hops did the destination MAC change? Why does the IP never change?

---

## Lab C — Packet Tracer: Multi-Subnet IP Design Build
**⏱️ ~40 min · Objective:** Design an IP scheme from scratch and build a routed, multi-subnet network. *This is the foundation lab for every later session.*

**Scenario:** One company, base network `192.168.0.0/24`, three departments:
Engineering (50 hosts), Marketing (25 hosts), Management (10 hosts).

### Step C1 — Subnet on paper (do this first!)
Pick the smallest subnet that fits each department (size = next power of two above *hosts + 2*).

| Dept | Hosts needed | CIDR | Mask | Network | Usable Range | Broadcast | Gateway |
|------|-------------|------|------|---------|--------------|-----------|---------|
| Engineering | 50 | `/26` | `255.255.255.192` | `192.168.0.0` | `.1 – .62` | `.63` | `192.168.0.1` |
| Marketing | 25 | `/27` | `255.255.255.224` | `192.168.0.64` | `.65 – .94` | `.95` | `192.168.0.65` |
| Management | 10 | `/28` | `255.255.255.240` | `192.168.0.96` | `.97 – .110` | `.111` | `192.168.0.97` |

> [!TIP]
> Check your math with the [quick formula](./README.md#how-to-subnet-step-by-step): `/26` → size `256/2² = 64`, usable `64 − 2 = 62`. ✅

📸 *Picture: your handwritten / spreadsheet subnet table.*
<!-- ![Subnet plan](./img/labC-step1-subnetplan.png) -->

### Step C2 — Build the topology
Drag on: **6 PCs** (2 per dept), **3 switches** (one per dept), **1 router**.
Cable each department's PCs to its own switch, then each switch to the router.

```
[ENG]  PC1 PC2 ── SW-Eng ─┐
[MKT]  PC3 PC4 ── SW-Mkt ─┼── Router
[MGT]  PC5 PC6 ── SW-Mgt ─┘
```

📸 *Picture: the three-department topology.*
<!-- ![Lab C topology](./img/labC-step2-topology.png) -->

### Step C3 — Configure the router (sub-interfaces / Router-on-a-Stick)
We give the router one logical gateway per subnet using **802.1Q sub-interfaces** on a trunk link. First tag each department on its switch with a VLAN (ENG=10, MKT=20, MGT=30) and set the switch uplink to **trunk**; then configure the router:

```
Router> enable
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# no shutdown
Router(config-if)# exit

! Engineering — VLAN 10
Router(config)# interface g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.0.1 255.255.255.192

! Marketing — VLAN 20
Router(config)# interface g0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.0.65 255.255.255.224

! Management — VLAN 30
Router(config)# interface g0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.0.97 255.255.255.240
Router(config-subif)# end
```

> [!NOTE]
> **Simpler alternative** if you haven't covered VLANs yet: give the router **3 separate physical interfaces** (g0/0, g0/1, g0/2), one per switch, each with the gateway IP above — no VLANs or trunk needed. VLANs are covered fully in Session 4.

📸 *Picture: router CLI after entering the sub-interfaces.*
<!-- ![Router config](./img/labC-step3-router.png) -->

### Step C4 — Assign PC IPs
Give each PC an address **inside its subnet's usable range**, pointing at its gateway:

| PC | Dept | IP | Mask | Gateway |
|----|------|------|------|---------|
| PC1 | ENG | `192.168.0.10` | `255.255.255.192` | `192.168.0.1` |
| PC2 | ENG | `192.168.0.11` | `255.255.255.192` | `192.168.0.1` |
| PC3 | MKT | `192.168.0.70` | `255.255.255.224` | `192.168.0.65` |
| PC4 | MKT | `192.168.0.71` | `255.255.255.224` | `192.168.0.65` |
| PC5 | MGT | `192.168.0.100` | `255.255.255.240` | `192.168.0.97` |
| PC6 | MGT | `192.168.0.101` | `255.255.255.240` | `192.168.0.97` |

📸 *Picture: one PC's IP Configuration filled in.*
<!-- ![PC addressing](./img/labC-step4-pcip.png) -->

### Step C5 — Verify connectivity
Open a PC's **Command Prompt** (Desktop tab) and test:

1. **Same subnet** — `ping 192.168.0.11` from PC1 → should reply immediately.
2. **Across subnets** — `ping 192.168.0.70` (Marketing) from PC1 → should reply via the router.
3. On the router, confirm every interface is `up/up`:
   ```
   Router# show ip interface brief
   ```

📸 *Picture: successful cross-subnet ping + `show ip interface brief` output.*
<!-- ![Verification](./img/labC-step5-verify.png) -->

### Step C6 — Break it on purpose (learning moments)
1. **Wrong mask:** set PC1's mask to `255.255.255.0` and re-ping Marketing → it now thinks Marketing is local, skips the gateway, and **fails**. Explain why, then fix it.
2. **IP conflict:** give PC2 the same IP as PC1 (`192.168.0.10`) → observe the conflict warning. Discuss why **IPAM** matters in real networks.

📸 *Picture: the failed ping and/or the IP-conflict popup.*
<!-- ![Failure demo](./img/labC-step6-failure.png) -->

> [!IMPORTANT]
> **Save your `.pkt` file** — you'll extend this topology in later sessions.

---

## Self-Check & Submission

Tick every box before you call S2 done:

- [ ] **Lab A:** captured and labelled a real `SYN / SYN-ACK / ACK` sequence
- [ ] **Lab A:** explained one difference between the TCP and UDP (DNS) conversations
- [ ] **Lab B:** stepped a ping through Simulation Mode and identified where the **MAC changes but the IP doesn't**
- [ ] **Lab C:** subnet table calculated **on paper** before building
- [ ] **Lab C:** same-subnet **and** cross-subnet pings succeed
- [ ] **Lab C:** reproduced and explained both the **wrong-mask** and **IP-conflict** failures
- [ ] All screenshots saved into `S2/img/` and rendering in this page

**Homework (from the README):** complete the [`10.0.0.0/16` 5-department design](./README.md#-homework) and build it in Packet Tracer.

---

## Is This Enough Practice for S2?

**Short answer: yes for the core objectives, with two suggested top-ups.**

✅ **Well covered by Labs A–C:**
- Reading real protocol headers (Ethernet / IP / TCP / UDP) — Lab A + Lab B
- TCP vs. UDP behaviour (handshake vs. connectionless) — Lab A
- Encapsulation and ARP hop-by-hop — Lab B
- Subnet *design from scratch*, building it, and verifying routing — Lab C
- The two most common real-world failures (wrong mask, IP conflict) — Lab C

🔹 **Recommended additions for stronger retention** (each ~10–15 min, optional):
1. **Subnetting drills without a calculator.** The labs build *one* scheme; fluency comes from repetition. Do 5–10 quick "given a CIDR, find network/broadcast/range" problems. The [homework `10.0.0.0/16` exercise](./README.md#-homework) is exactly this — treat it as required, not optional.
2. **A second handshake variant.** In Lab A, also capture a **TCP teardown** (`tcp.flags.fin == 1`) so students see `FIN/ACK` as well as `SYN`, and an **HTTPS** session to note that the payload is encrypted but the handshake is still visible.

🔸 **Deliberately deferred to later sessions (don't add here):**
- VLAN theory & trunking → **Session 4** (Lab C only uses them as a means to an end)
- DHCP/DNS/ARP services in depth → **Session 3**
- Routing protocols → **Session 4**

**Verdict:** Labs A–C fully exercise every Session 2 learning objective. Add the subnetting drills (via the homework) for fluency, and the optional FIN/HTTPS capture for depth — then S2 practice is complete.
