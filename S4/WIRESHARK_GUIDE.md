## Wireshark — Lab B: VLAN & Routing Traffic Analysis

> Companion guide to [Session 4 — Routing, Switching & VLANs](./README.md).
> Read this **before** doing **Lab B** in the [README](./README.md#hands-on-lab).
> New to Wireshark? See the [Session 1 guide](../S1/WIRESHARK_GUIDE.md) for installation and the basics, and the [Session 2 guide](../S2/WIRESHARK_GUIDE.md) for filters & Follow-Stream.

---

- [Wireshark — Lab B: VLAN \& Routing Traffic Analysis](#wireshark--lab-b-vlan--routing-traffic-analysis)
  - [Background: the 802.1Q VLAN tag](#background-the-8021q-vlan-tag)
  - [Lab B — Analyse VLAN & Routed Traffic](#lab-b--analyse-vlan--routed-traffic)
    - [Step B1 — Open the tagged capture](#step-b1--open-the-tagged-capture)
    - [Step B2 — Find the VLAN IDs](#step-b2--find-the-vlan-ids)
    - [Step B3 — Inspect the 802.1Q tag in the Ethernet header](#step-b3--inspect-the-8021q-tag-in-the-ethernet-header)
    - [Step B4 — Trace a ping across subnets (TTL)](#step-b4--trace-a-ping-across-subnets-ttl)
    - [Step B5 — Map subnets with Endpoints](#step-b5--map-subnets-with-endpoints)
    - [Step B6 — Map flows with Conversations](#step-b6--map-flows-with-conversations)
  - [Lab B Questions](#lab-b-questions)
  - [Next Steps](#next-steps)

---

### Background: the 802.1Q VLAN tag

When a frame crosses a **trunk** (the switch-to-switch and switch-to-router links you built in [Packet Tracer Lab A](./PACKET_TRACER_GUIDE.md)), the switch inserts a **4-byte 802.1Q tag** into the Ethernet header so the far end knows which **VLAN** the frame belongs to. The tag sits right after the source MAC and contains:

| Field | Size | Meaning |
|:---|:---|:---|
| **TPID** | 16 bits | Tag Protocol ID — always **`0x8100`**, the flag that says "an 802.1Q tag follows." |
| **PCP** | 3 bits | Priority (QoS class). |
| **DEI** | 1 bit | Drop-eligible indicator. |
| **VLAN ID** | 12 bits | The **VLAN number** (1–4094) — e.g. 10, 20, 30. |

Access ports strip this tag; trunk ports carry it. This lab reads those tags straight out of a capture.

---

## Lab B — Analyse VLAN & Routed Traffic

**⏱️ ~25 min · Objective:** find VLAN tags in real frames, watch TTL drop at each router hop, and map subnets/flows with Wireshark statistics.

### Step B1 — Open the tagged capture

Open the provided PCAP that contains **802.1Q-tagged** traffic: **File → Open** → select the `.pcap`/`.pcapng`. *(No trace file? Capture on a trunk link, or export one from your Lab A build.)*

<p align="center">
  <!-- ![Open capture](./img/labB-step1-open.png) -->
  <em>Fig. 1 — 📸 <code>img/labB-step1-open.png</code>: the tagged capture loaded in Wireshark.</em>
</p>

### Step B2 — Find the VLAN IDs

In the filter bar type **`vlan`** and press **Enter** — only 802.1Q-tagged frames remain. Note the different **VLAN IDs** present. To see one specific VLAN, use e.g. **`vlan.id == 20`**.

<p align="center">
  <!-- ![VLAN filter](./img/labB-step2-vlanfilter.png) -->
  <em>Fig. 2 — 📸 <code>img/labB-step2-vlanfilter.png</code>: the <code>vlan</code> filter showing tagged frames and their IDs.</em>
</p>

### Step B3 — Inspect the 802.1Q tag in the Ethernet header

Select a tagged frame and expand **Ethernet II → 802.1Q Virtual LAN** in the details pane. Confirm the **TPID = `0x8100`**, read the **Priority** and **ID** fields, and note where the tag sits in the raw bytes.

<p align="center">
  <!-- ![802.1Q tag](./img/labB-step3-tag.png) -->
  <em>Fig. 3 — 📸 <code>img/labB-step3-tag.png</code>: the 802.1Q tag expanded — TPID <code>0x8100</code> and the VLAN ID.</em>
</p>

### Step B4 — Trace a ping across subnets (TTL)

1. Filter **`icmp`** and find an echo request/reply that crosses **between subnets** (different source/destination networks).
2. Expand **Internet Protocol Version 4 → Time to Live** and compare the TTL at the start versus after the router. **TTL drops by 1 at each router hop** — that's how you count hops.

<p align="center">
  <!-- ![ICMP TTL](./img/labB-step4-ttl.png) -->
  <em>Fig. 4 — 📸 <code>img/labB-step4-ttl.png</code>: the IPv4 TTL field, decremented by the router.</em>
</p>

### Step B5 — Map subnets with Endpoints

Open **Statistics → Endpoints → IPv4**. This lists every IP that appears, letting you spot the **distinct subnets** (e.g. `192.168.10.x`, `192.168.20.x`, `192.168.30.x`) active in the capture.

<p align="center">
  <!-- ![Endpoints](./img/labB-step5-endpoints.png) -->
  <em>Fig. 5 — 📸 <code>img/labB-step5-endpoints.png</code>: Statistics → Endpoints listing the active subnets.</em>
</p>

### Step B6 — Map flows with Conversations

Open **Statistics → Conversations → IPv4**. Each row is a **flow between two IPs** — use it to see which VLANs/subnets are talking to each other and how much traffic crosses the router between them.

<p align="center">
  <!-- ![Conversations](./img/labB-step6-conversations.png) -->
  <em>Fig. 6 — 📸 <code>img/labB-step6-conversations.png</code>: Statistics → Conversations mapping inter-VLAN traffic.</em>
</p>

---

## Lab B Questions

**Try each one first, then click "Show answer".**

**Q1.** What value identifies an **802.1Q tag**, and where in the frame does it sit?

<details>
<summary>💡 Show answer</summary>

The **TPID = `0x8100`** marks an 802.1Q tag. It sits in the **Ethernet header, right after the source MAC address** (where the EtherType would normally be). The real EtherType then follows the 4-byte tag. The tag's lower 12 bits are the **VLAN ID**.
</details>

**Q2.** You see the **same VLAN tag on a switch-to-switch link** but **no tag** on the link to a PC. Why?

<details>
<summary>💡 Show answer</summary>

The switch-to-switch link is a **trunk**, which **tags** every frame so the other switch knows the VLAN. The link to a PC is an **access port**, which **strips the tag** — end devices don't understand (or need) VLAN tags; they just see plain Ethernet.
</details>

**Q3.** A ping crossed the network and the **TTL dropped from 128 to 126**. How many routers did it pass through?

<details>
<summary>💡 Show answer</summary>

**Two.** TTL is decremented by **1 per router hop**, so 128 → 126 means **2 hops/routers**. (Switches don't decrement TTL — they operate at Layer 2 and leave the IP header untouched.)
</details>

**Q4.** What's the difference between **Statistics → Endpoints** and **Statistics → Conversations**?

<details>
<summary>💡 Show answer</summary>

**Endpoints** lists each **individual address** that appears (who's on the network — great for spotting subnets/VLANs). **Conversations** lists each **pair of addresses talking** (who's talking to whom, and how much) — great for mapping inter-VLAN flows and spotting heavy talkers.
</details>

---

## Next Steps

- Cross-check against your build: the VLAN IDs and subnets you see here should match the VLAN plan from [Packet Tracer Lab A](./PACKET_TRACER_GUIDE.md).
- **Homework (from the README):** write a paragraph on **OSPF vs static routing**, and try **port security** on an access port (max 1 MAC).
- Compare with earlier captures: the **TTL** mechanic here is the same one behind `ping`/`traceroute` from [Session 1](../S1/README.md#deep-dive-the-ping-command--options).
