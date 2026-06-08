## Cisco Packet Tracer — Labs B & C: Protocol Walk + Multi-Subnet Design

> Companion guide to [Session 2 — The Protocol Stack Deep Dive](./README.md).
> Read this **before** doing **Lab B** and **Lab C** in the [README](./README.md#️-hands-on-lab).
> New to Packet Tracer? Start with the [Session 1 Packet Tracer guide](../S1/PACKET_TRACER_GUIDE.md) for installation, the window tour, and CLI basics.

---

- [Cisco Packet Tracer — Labs B \& C](#cisco-packet-tracer--labs-b--c-protocol-walk--multi-subnet-design)
  - [🧪 Lab B — Simulation-Mode Protocol Walk](#-lab-b--simulation-mode-protocol-walk)
    - [Step B1 — Build the topology](#step-b1--build-the-topology)
    - [Step B2 — Address the devices](#step-b2--address-the-devices)
    - [Step B3 — Enter Simulation Mode](#step-b3--enter-simulation-mode)
    - [Step B4 — Send a ping and step through](#step-b4--send-a-ping-and-step-through)
    - [Step B5 — Inspect the PDU layers](#step-b5--inspect-the-pdu-layers)
    - [📝 Lab B Questions](#-lab-b-questions)
  - [🧪 Lab C — Multi-Subnet IP Design Build](#-lab-c--multi-subnet-ip-design-build)
    - [Step C1 — Subnet on paper (do this first!)](#step-c1--subnet-on-paper-do-this-first)
    - [Step C2 — Build the topology](#step-c2--build-the-topology)
    - [Step C3 — Configure the router](#step-c3--configure-the-router)
    - [Step C4 — Assign PC IPs](#step-c4--assign-pc-ips)
    - [Step C5 — Verify connectivity](#step-c5--verify-connectivity)
    - [Step C6 — Break it on purpose](#step-c6--break-it-on-purpose)
    - [📝 Lab C Questions](#-lab-c-questions)
  - [✅ Self-Check & Submission](#-self-check--submission)
  - [➡️ Next Steps](#️-next-steps)

---

## 🧪 Lab B — Simulation-Mode Protocol Walk

**⏱️ ~20 min · Objective:** watch encapsulation (**ICMP → IP → Ethernet**) and **ARP** resolution happen *hop-by-hop*.

### Step B1 — Build the topology

Drag the devices onto the workspace and cable them with **Copper Straight-Through**:

```text
PC1 ──┐
      ├── Switch ── Router ── Server
PC2 ──┘
```

* PCs → Switch: Copper Straight-Through
* Switch → Router: Copper Straight-Through
* Router → Server: Copper Straight-Through (or place the Server on a second switch)

<p align="center">
  <!-- ![Lab B topology](./img/labB-step1-topology.png) -->
  <em>Fig. 1 — 📸 <code>img/labB-step1-topology.png</code>: the finished topology on the workspace.</em>
</p>

### Step B2 — Address the devices

Put the PCs and the Server on **different subnets** so traffic *must* cross the router.

| Device | IP | Mask | Gateway |
|:---|:---|:---|:---|
| PC1 | `192.168.1.10` | `255.255.255.0` | `192.168.1.1` |
| PC2 | `192.168.1.11` | `255.255.255.0` | `192.168.1.1` |
| Server | `192.168.2.10` | `255.255.255.0` | `192.168.2.1` |

Configure the router interfaces (CLI tab):

```ios
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

> 💡 Need a refresher on `enable`, `configure terminal`, and what `ip address … 255.255.255.0` means? See the [Session 1 router-config explainer](../S1/PACKET_TRACER_GUIDE.md#step-5--configure-the-router).

<p align="center">
  <!-- ![Addressing](./img/labB-step2-addressing.png) -->
  <em>Fig. 2 — 📸 <code>img/labB-step2-addressing.png</code>: PC IP Configuration + router CLI showing interfaces up.</em>
</p>

### Step B3 — Enter Simulation Mode

1. Click **Simulation** (bottom-right corner) to pause real-time traffic.
2. In **Edit Filters**, keep only **ICMP** and **ARP** checked to cut the noise.

<p align="center">
  <!-- ![Simulation mode](./img/labB-step3-simulation.png) -->
  <em>Fig. 3 — 📸 <code>img/labB-step3-simulation.png</code>: the Simulation panel with ICMP + ARP filters selected.</em>
</p>

### Step B4 — Send a ping and step through

1. Use the **Add Simple PDU** (envelope) tool: click **PC1**, then the **Server**.
2. Click **Play / Capture-Forward** repeatedly to advance **one hop at a time**.
3. Watch the sequence:
   * **ARP request/reply** — PC1 learns the router's MAC (and the router learns the Server's).
   * **ICMP echo** travelling *inside* IP *inside* Ethernet.

<p align="center">
  <!-- ![Ping stepping](./img/labB-step4-ping.png) -->
  <em>Fig. 4 — 📸 <code>img/labB-step4-ping.png</code>: a packet mid-flight between switch and router.</em>
</p>

### Step B5 — Inspect the PDU layers

1. Click a coloured packet square → the **PDU Information** window opens.
2. Use the **OSI Model** tab to read each layer; use **Inbound/Outbound PDU Details** to see the real header fields (Ethernet → IP → ICMP).
3. Confirm the key idea of routing: the **destination MAC changes at the router**, but the **destination IP stays the same** end-to-end.

<p align="center">
  <!-- ![PDU inspection](./img/labB-step5-pdu.png) -->
  <em>Fig. 5 — 📸 <code>img/labB-step5-pdu.png</code>: PDU Details showing the Ethernet + IP + ICMP headers.</em>
</p>

### 📝 Lab B Questions

**Try each one first, then click "Show answer".**

**Q1.** Before the very first ICMP echo could be sent, an **ARP** exchange happened. What was PC1 actually asking for, and why?

<details>
<summary>💡 Show answer</summary>

PC1 needed the **MAC address of its default gateway** (the router's `192.168.1.1`). It already knows the *IP* it wants to reach, but to build the **Ethernet frame** it must fill in a **destination MAC** — and on its own subnet that has to be the gateway's MAC (since the Server is on a different network). ARP is the "what MAC owns this IP?" broadcast that resolves it.
</details>

**Q2.** As the ping crossed the router, the **destination MAC changed** but the **destination IP did not**. Why?

<details>
<summary>💡 Show answer</summary>

**IP addresses identify the original source and final destination** — they stay constant for the whole journey. **MAC addresses only identify devices on the current physical link**, so they're rewritten at every hop: PC1→router uses the router's MAC; router→Server uses the Server's MAC. "IP stays, MAC changes per hop" is the heart of how routing works.
</details>

**Q3.** In the PDU's **OSI Model** tab, which layers does the **switch** process versus the **router**?

<details>
<summary>💡 Show answer</summary>

The **switch** is a Layer-2 device — it only looks at **Layer 2 (Ethernet/MAC)** to forward the frame, leaving Layers 3+ untouched. The **router** goes up to **Layer 3 (IP)** to read the destination IP, decide the next hop, decrement TTL, then re-encapsulate down to Layer 2 on the way out.
</details>

---

## 🧪 Lab C — Multi-Subnet IP Design Build

**⏱️ ~40 min · Objective:** design an IP scheme **from scratch** and build a routed, multi-subnet network. *This is the foundation lab for every later session.*

**Scenario:** one company, base network `192.168.0.0/24`, three departments:
**Engineering** (50 hosts), **Marketing** (25 hosts), **Management** (10 hosts).

### Step C1 — Subnet on paper (do this first!)

Pick the smallest subnet that fits each department (size = next power of two **above** *hosts + 2*, to leave room for the network and broadcast addresses).

| Dept | Hosts needed | CIDR | Mask | Network | Usable Range | Broadcast | Gateway |
|:---|:---:|:---:|:---|:---|:---|:---|:---|
| Engineering | 50 | `/26` | `255.255.255.192` | `192.168.0.0` | `.1 – .62` | `.63` | `192.168.0.1` |
| Marketing | 25 | `/27` | `255.255.255.224` | `192.168.0.64` | `.65 – .94` | `.95` | `192.168.0.65` |
| Management | 10 | `/28` | `255.255.255.240` | `192.168.0.96` | `.97 – .110` | `.111` | `192.168.0.97` |

> 💡 Check your math with the [quick formula](./README.md#how-to-subnet-step-by-step): `/26` → size `256 / 2² = 64`, usable `64 − 2 = 62`. ✅

<p align="center">
  <!-- ![Subnet plan](./img/labC-step1-subnetplan.png) -->
  <em>Fig. 6 — 📸 <code>img/labC-step1-subnetplan.png</code>: your handwritten / spreadsheet subnet table.</em>
</p>

### Step C2 — Build the topology

Drag on **6 PCs** (2 per dept), **3 switches** (one per dept), and **1 router** with **three Ethernet ports** — a Cisco **2911** works well (it has `g0/0`, `g0/1`, `g0/2` on-board). Cable each department's PCs to its own switch, then connect **each switch to its own router port** (one physical interface per subnet).

```text
[ENG]  PC1 PC2 ── SW-Eng ─┐
[MKT]  PC3 PC4 ── SW-Mkt ─┼── Router
[MGT]  PC5 PC6 ── SW-Mgt ─┘
```

<p align="center">
  <!-- ![Lab C topology](./img/labC-step2-topology.png) -->
  <em>Fig. 7 — 📸 <code>img/labC-step2-topology.png</code>: the three-department topology.</em>
</p>

### Step C3 — Configure the router

Give the router **one physical interface per subnet** — each becomes that department's **gateway**. This keeps the focus on *subnetting*: one interface, one network, one gateway IP. Assign each department's gateway address and bring the interface up with `no shutdown`:

```ios
Router> enable
Router# configure terminal

! Engineering — 192.168.0.0/26
Router(config)# interface g0/0
Router(config-if)# ip address 192.168.0.1 255.255.255.192
Router(config-if)# no shutdown
Router(config-if)# exit

! Marketing — 192.168.0.64/27
Router(config)# interface g0/1
Router(config-if)# ip address 192.168.0.65 255.255.255.224
Router(config-if)# no shutdown
Router(config-if)# exit

! Management — 192.168.0.96/28
Router(config)# interface g0/2
Router(config-if)# ip address 192.168.0.97 255.255.255.240
Router(config-if)# no shutdown
Router(config-if)# end
```

> [!NOTE]
> Notice each interface's **mask matches its department's subnet size** (`/26`, `/27`, `/28`) — the same numbers from your Step C1 plan. No VLANs or trunking here; that's a Session 4 topic. One subnet = one router port = one gateway.

<p align="center">
  <!-- ![Router config](./img/labC-step3-router.png) -->
  <em>Fig. 8 — 📸 <code>img/labC-step3-router.png</code>: router CLI after configuring the three interfaces.</em>
</p>

### Step C4 — Assign PC IPs

Give each PC an address **inside its subnet's usable range**, pointing at its gateway:

| PC | Dept | IP | Mask | Gateway |
|:---|:---:|:---|:---|:---|
| PC1 | ENG | `192.168.0.10` | `255.255.255.192` | `192.168.0.1` |
| PC2 | ENG | `192.168.0.11` | `255.255.255.192` | `192.168.0.1` |
| PC3 | MKT | `192.168.0.70` | `255.255.255.224` | `192.168.0.65` |
| PC4 | MKT | `192.168.0.71` | `255.255.255.224` | `192.168.0.65` |
| PC5 | MGT | `192.168.0.100` | `255.255.255.240` | `192.168.0.97` |
| PC6 | MGT | `192.168.0.101` | `255.255.255.240` | `192.168.0.97` |

<p align="center">
  <!-- ![PC addressing](./img/labC-step4-pcip.png) -->
  <em>Fig. 9 — 📸 <code>img/labC-step4-pcip.png</code>: one PC's IP Configuration filled in.</em>
</p>

### Step C5 — Verify connectivity

Open a PC's **Command Prompt** (Desktop tab) and test:

1. **Same subnet** — `ping 192.168.0.11` from PC1 → should reply immediately.
2. **Across subnets** — `ping 192.168.0.70` (Marketing) from PC1 → should reply via the router.
3. On the router, confirm every interface is `up/up`:
   ```ios
   Router# show ip interface brief
   ```

<p align="center">
  <!-- ![Verification](./img/labC-step5-verify.png) -->
  <em>Fig. 10 — 📸 <code>img/labC-step5-verify.png</code>: a successful cross-subnet ping + <code>show ip interface brief</code> output.</em>
</p>

### Step C6 — Break it on purpose

Failures teach faster than successes — reproduce these two, then fix them:

1. **Wrong mask:** set PC1's mask to `255.255.255.0` and re-ping Marketing → it now thinks Marketing is *local*, skips the gateway, and **fails**. Explain why, then restore `/26`.
2. **IP conflict:** give PC2 the same IP as PC1 (`192.168.0.10`) → observe the conflict warning. Discuss why **IPAM** (IP Address Management) matters in real networks.

<p align="center">
  <!-- ![Failure demo](./img/labC-step6-failure.png) -->
  <em>Fig. 11 — 📸 <code>img/labC-step6-failure.png</code>: the failed ping and/or the IP-conflict popup.</em>
</p>

> [!IMPORTANT]
> **Save your `.pkt` file** — you'll extend this topology in later sessions.

### 📝 Lab C Questions

**Try each one first, then click "Show answer".**

**Q1.** Engineering needs **50 hosts**. Why is `/26` the right choice and not `/27` or `/25`?

<details>
<summary>💡 Show answer</summary>

A subnet loses 2 addresses (network + broadcast), so usable = size − 2. **`/27`** = 32 addresses → only **30 usable** — too few for 50. **`/26`** = 64 addresses → **62 usable** — fits 50 with room to grow. **`/25`** = 126 usable would also fit, but it **wastes** addresses and would overlap the space the other departments need. `/26` is the smallest block that fits — the efficient choice.
</details>

**Q2.** After setting PC1's mask to `255.255.255.0` (Step C6), why did the ping to Marketing **fail**?

<details>
<summary>💡 Show answer</summary>

With a `/24` mask, PC1 believes the **whole `192.168.0.0/24`** is its *local* subnet — including Marketing's `192.168.0.70`. So instead of sending the packet to its **gateway** (the router) to be routed, it tries to reach Marketing **directly with ARP** on its own segment. Nothing answers (Marketing is physically on a different switch/subnet), so the ping fails. The mask, not the IP, is what told PC1 "this is local."
</details>

**Q3.** What does `show ip interface brief` tell you, and what should the **Status / Protocol** columns read for a working link?

<details>
<summary>💡 Show answer</summary>

It lists every router interface with its **IP address** and two state columns. A healthy, configured interface shows **`up / up`** (line protocol up, layer-1 up). `administratively down` means you forgot **`no shutdown`**; `up / down` usually means a cabling or encapsulation mismatch.
</details>

**Q4.** Why is doing the subnet table **on paper before building** strongly recommended?

<details>
<summary>💡 Show answer</summary>

Because the addressing plan drives **everything** — gateway IPs, masks, and which PC belongs where. Designing it first prevents overlapping subnets and mid-build rework; in Packet Tracer (and real life) fixing an address scheme *after* you've configured 10 devices is far more painful than getting the table right up front. It's the same discipline real network engineers use (IPAM).
</details>

---

## ✅ Self-Check & Submission

Tick every box before you call Session 2 done:

- [ ] **Lab A:** captured and labelled a real `SYN / SYN-ACK / ACK` sequence *(see the [Wireshark guide](./WIRESHARK_GUIDE.md))*
- [ ] **Lab A:** explained one difference between the TCP and UDP (DNS) conversations
- [ ] **Lab B:** stepped a ping through Simulation Mode and identified where the **MAC changes but the IP doesn't**
- [ ] **Lab C:** subnet table calculated **on paper** before building
- [ ] **Lab C:** same-subnet **and** cross-subnet pings succeed
- [ ] **Lab C:** reproduced and explained both the **wrong-mask** and **IP-conflict** failures
- [ ] All screenshots saved into [`S2/img/`](./img/) and rendering in these guides

---

## ➡️ Next Steps

- **Homework (from the README):** complete the [`10.0.0.0/16` 5-department design](./README.md#-homework) and build it in Packet Tracer — this is the subnetting *repetition* that turns the one-off Lab C into fluency.
- Keep your `.pkt` file; later sessions extend this multi-subnet topology with DHCP/DNS (Session 3) and routing/VLANs (Session 4).
- Revisit the [Wireshark guide](./WIRESHARK_GUIDE.md) and capture a **TCP teardown** (`tcp.flags.fin == 1`) for a complete picture of a connection's life cycle.
