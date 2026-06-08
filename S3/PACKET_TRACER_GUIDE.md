## Cisco Packet Tracer — Lab B: DHCP & DNS Server Setup

> Companion guide to [Session 3 — Network Services & Security](./README.md).
> Read this **before** doing **Lab B** in the [README](./README.md#️-hands-on-lab).
> New to Packet Tracer? Start with the [Session 1 Packet Tracer guide](../S1/PACKET_TRACER_GUIDE.md) for installation, the window tour, and CLI basics.

---

- [Cisco Packet Tracer — Lab B: DHCP \& DNS Server Setup](#cisco-packet-tracer--lab-b-dhcp--dns-server-setup)
  - [🧩 What You'll Build](#-what-youll-build)
  - [🧪 Lab B — DHCP & DNS Server Setup](#-lab-b--dhcp--dns-server-setup)
    - [Step B1 — Build the topology](#step-b1--build-the-topology)
    - [Step B2 — Configure the router's DHCP pool](#step-b2--configure-the-routers-dhcp-pool)
    - [Step B3 — Set the PCs to DHCP](#step-b3--set-the-pcs-to-dhcp)
    - [Step B4 — Configure the DNS server](#step-b4--configure-the-dns-server)
    - [Step B5 — Watch DORA in Simulation Mode](#step-b5--watch-dora-in-simulation-mode)
    - [Step B6 — Resolve a name from a PC](#step-b6--resolve-a-name-from-a-pc)
  - [📝 Lab B Questions](#-lab-b-questions)
  - [✅ Self-Check](#-self-check)
  - [➡️ Next Steps](#️-next-steps)

---

### 🧩 What You'll Build

In [Wireshark Lab A](./WIRESHARK_GUIDE.md) you *watched* DHCP and DNS work. Here you build the **server side**: a router that hands out addresses (**DHCP**) and a server that resolves names (**DNS**). When you're done, a PC set to "obtain IP automatically" will get a full configuration on its own and reach a website by name — no manual addressing at all.

| Device | Role | Address |
|:---|:---|:---|
| Router | DHCP server (pool) + gateway | `192.168.1.1` |
| DNS Server | Resolves `www.lab.com` | `192.168.1.100` (static) |
| PC1 / PC2 / PC3 | Clients | obtained via DHCP |

---

## 🧪 Lab B — DHCP & DNS Server Setup

**⏱️ ~25 min · Objective:** configure DHCP + DNS so clients auto-configure and browse by name.

### Step B1 — Build the topology

Drag on **3 PCs**, **1 switch**, **1 router**, and **1 Server** (used as the DNS server). Cable everything with **Copper Straight-Through**:

```text
PC1 ─┐
PC2 ─┼── Switch ── Router
PC3 ─┘      │
         DNS Server (192.168.1.100)
```

Give the router's LAN interface the gateway IP, and the Server a **static** IP:

```ios
Router> enable
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
```

<p align="center">
  <!-- ![Lab B topology](./img/labB-step1-topology.png) -->
  <em>Fig. 1 — 📸 <code>img/labB-step1-topology.png</code>: the finished topology with router + DNS server.</em>
</p>

### Step B2 — Configure the router's DHCP pool

On the router, create a DHCP pool that gives clients an **address range, gateway, and DNS server**. Exclude the addresses you've assigned statically (the router and the DNS server):

```ios
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
Router(config)# ip dhcp excluded-address 192.168.1.100
Router(config)# ip dhcp pool LAN
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 192.168.1.100
Router(dhcp-config)# end
```

**What each line does.** The first two commands carve out addresses the pool must *not* give away; the rest define the pool itself:

| Command | What it does |
|:---|:---|
| `ip dhcp excluded-address 192.168.1.1 192.168.1.10` | **Reserves a range** (`.1`–`.10`) that DHCP will **never** lease — leaving room for statically-assigned gear (the router, switches, printers). A *range* takes two addresses: first and last. |
| `ip dhcp excluded-address 192.168.1.100` | **Reserves a single address** — the DNS server's static IP — so it can't be handed to a PC. (One address = one argument.) |
| `ip dhcp pool LAN` | **Creates the pool** named `LAN` and drops you into pool-config mode — notice the prompt changes to `Router(dhcp-config)#`. |
| `network 192.168.1.0 255.255.255.0` | The **range of addresses to lease** — the whole `192.168.1.0/24` subnet, minus the excluded ones. The mask also tells clients their subnet size. |
| `default-router 192.168.1.1` | The **gateway** address clients receive (DHCP Option 3). Without it, leased PCs couldn't leave their subnet. |
| `dns-server 192.168.1.100` | The **DNS resolver** address clients receive (DHCP Option 6). Without it, PCs get an IP but can't resolve names. |

> 💡 **`default-router`** and **`dns-server`** become the **Option fields** you read in the DHCP **ACK** in [Wireshark Lab A](./WIRESHARK_GUIDE.md#step-a3--find-the-4-dora-packets) — this is the server *side* of that exact exchange.
>
> ⚠️ **Order matters:** set the `excluded-address` ranges **before** clients request leases, or the router may already have handed out an address you meant to reserve. And note the **prompt change** — `network`/`default-router`/`dns-server` only work inside `Router(dhcp-config)#`, *after* `ip dhcp pool LAN`.

<p align="center">
  <!-- ![DHCP pool](./img/labB-step2-dhcppool.png) -->
  <em>Fig. 2 — 📸 <code>img/labB-step2-dhcppool.png</code>: the router CLI after defining the DHCP pool.</em>
</p>

### Step B3 — Set the PCs to DHCP

On each PC: **Desktop → IP Configuration → DHCP**. Within a moment the PC should show an address from the pool (e.g. `192.168.1.11`), plus the gateway and DNS server it was handed.

<p align="center">
  <!-- ![PC DHCP](./img/labB-step3-pcdhcp.png) -->
  <em>Fig. 3 — 📸 <code>img/labB-step3-pcdhcp.png</code>: a PC set to DHCP, showing the leased address.</em>
</p>

### Step B4 — Configure the DNS server

Click the **Server → Services tab → DNS**:

1. Turn the DNS service **On**.
2. Add an **A record**: Name `www.lab.com`, Address `192.168.1.100` (and/or point it at a web server you've enabled under **HTTP**).
3. Click **Add**.

<p align="center">
  <!-- ![DNS server](./img/labB-step4-dns.png) -->
  <em>Fig. 4 — 📸 <code>img/labB-step4-dns.png</code>: the DNS service On with the <code>www.lab.com</code> A record added.</em>
</p>

### Step B5 — Watch DORA in Simulation Mode

1. Click **Simulation** (bottom-right) and filter to **DHCP** (and DNS).
2. On a PC, toggle IP Configuration to **Static** then back to **DHCP** to trigger a fresh request.
3. Step through the **Discover → Offer → Request → ACK** packets and click each to inspect the layers.

<p align="center">
  <!-- ![DORA simulation](./img/labB-step5-dora.png) -->
  <em>Fig. 5 — 📸 <code>img/labB-step5-dora.png</code>: the DORA exchange stepping through Simulation Mode.</em>
</p>

### Step B6 — Resolve a name from a PC

1. Open a PC's **Desktop → Command Prompt** and run `ping www.lab.com` — it should resolve to `192.168.1.100` and reply.
2. Or open **Desktop → Web Browser** and visit **`www.lab.com`** — watch the **DNS query** resolve the name, then the **HTTP** page load.

<p align="center">
  <!-- ![Resolve name](./img/labB-step6-resolve.png) -->
  <em>Fig. 6 — 📸 <code>img/labB-step6-resolve.png</code>: a PC resolving and reaching <code>www.lab.com</code> by name.</em>
</p>

---

## 📝 Lab B Questions

**Try each one first, then click "Show answer".**

**Q1.** Why must the router's IP (`192.168.1.1`) and the DNS server's IP (`192.168.1.100`) be **excluded** from the DHCP pool?

<details>
<summary>💡 Show answer</summary>

Because they're assigned **statically**. If DHCP were free to lease those same addresses to a PC, you'd get an **IP conflict** — two devices claiming one address. `ip dhcp excluded-address` reserves them so the pool never hands them out.
</details>

**Q2.** A PC set to DHCP received an address **and** a gateway **and** a DNS server. Where did each value come from?

<details>
<summary>💡 Show answer</summary>

All three come from the pool definition in the **DHCP ACK**: the **address** from `network`, the gateway from **`default-router`**, and the resolver from **`dns-server`**. This is exactly the Option data you read in the Wireshark ACK in Lab A.
</details>

**Q3.** When the PC browsed to `www.lab.com`, two different protocols ran in order. What were they and why?

<details>
<summary>💡 Show answer</summary>

First a **DNS** query resolved `www.lab.com` → `192.168.1.100` (you can't connect to a *name*, only an IP). Then **HTTP** opened a connection to that IP to fetch the page. Name-resolution always precedes the actual data connection.
</details>

**Q4.** If you **forgot** the `dns-server` line in the pool, the PC could still `ping 192.168.1.100` but **not** `ping www.lab.com`. Why?

<details>
<summary>💡 Show answer</summary>

Pinging the **IP** needs no name resolution, so it works. Pinging the **name** requires DNS — but without a DNS server address, the PC has no resolver to ask, so the name lookup fails. It proves connectivity and name-resolution are two separate things.
</details>

---

## ✅ Self-Check

- [ ] Router LAN interface up at `192.168.1.1`; DHCP pool defined with `network`, `default-router`, `dns-server`
- [ ] All 3 PCs obtained an address **via DHCP** (not typed in by hand)
- [ ] DNS service **On** with a working `www.lab.com` A record
- [ ] Stepped the **DORA** exchange in Simulation Mode
- [ ] A PC reached **`www.lab.com` by name** (DNS query → HTTP)
- [ ] Screenshots saved into [`S3/img/`](./img/) and rendering in this guide

---

## ➡️ Next Steps

- Pair this with [Wireshark Labs A & C](./WIRESHARK_GUIDE.md): capture the very DORA and DNS packets your server here produces, then look at the **security** angle (rogue DHCP, DNS spoofing).
- **Homework (from the README):** the Kurose & Ross **DNS** Wireshark lab + a paragraph on **DNS poisoning/spoofing**.
- Save your `.pkt` file — later sessions add routing and security hardening (DHCP snooping, DAI) on top of these services.
