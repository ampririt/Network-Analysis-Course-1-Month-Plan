## Wireshark — Lab A: DHCP & DNS Analysis

> Companion guide to [Session 3 — Network Services & Security](./README.md).
> Read this **before** doing **Lab A** in the [README](./README.md#hands-on-lab).
> New to Wireshark? Start with the [Session 1 Wireshark guide](../S1/WIRESHARK_GUIDE.md) for installation and the basics, and the [Session 2 guide](../S2/WIRESHARK_GUIDE.md) for filters & Follow-Stream.

---

- [Wireshark — Lab A: DHCP \& DNS Analysis](#wireshark--lab-a-dhcp--dns-analysis)
- [Lab A — DHCP \& DNS Analysis](#lab-a--dhcp--dns-analysis)
  - [Background: DORA \& name resolution](#background-dora--name-resolution)
  - [Step A1 — Gather a DHCP trace](#step-a1--gather-a-dhcp-trace)
  - [Step A2 — Read the four DORA messages](#step-a2--read-the-four-dora-messages)
  - [Step A3 — Explore names with nslookup](#step-a3--explore-names-with-nslookup)
  - [Step A4 — Capture DNS while browsing](#step-a4--capture-dns-while-browsing)
  - [Step A5 — Inspect the DNS query \& response](#step-a5--inspect-the-dns-query--response)
  - [Lab A Questions](#lab-a-questions)
- [Next Steps](#next-steps)

---

## Lab A — DHCP & DNS Analysis

**Objective:** capture a device **getting an address (DHCP/DORA)** and **resolving names (DNS)** — the two services that make "the network just work."

> *Adapted from "Wireshark Lab: DHCP v9" and "Wireshark Lab: DNS v9.0", supplements to* Computer Networking: A Top-Down Approach*, J.F. Kurose & K.W. Ross.*

### Background: DORA & name resolution

When a device joins a network it has no IP yet, so it asks for one. **DHCP** (Dynamic Host Configuration Protocol) answers in a four-step exchange remembered as **DORA**:

| # | Message | From → To | Purpose |
|:---:|:---|:---|:---|
| 1 | **D**iscover | Client → broadcast | "Is there a DHCP server out there?" |
| 2 | **O**ffer | Server → client | "Yes — here's an address you can have." |
| 3 | **R**equest | Client → broadcast | "I'll take that offered address." |
| 4 | **A**ck | Server → client | "It's yours — here's your lease, gateway, and DNS server." |

> Only **Request** and **ACK** are *mandatory* — Discover/Offer can be skipped when a host simply **renews** an address it already had. All four messages of one exchange share a single **Transaction ID (xid)**, which is how the client matches the Offer/ACK to its own Discover/Request.

DHCP runs over **UDP** — server port **67**, client port **68**. Once addressed, the device uses **DNS** (Domain Name System, **UDP port 53**) to turn names like `gaia.cs.umass.edu` into IP addresses via a **query → response** to its **local DNS server**.

---

**Part 1 — DHCP: watch a host get an address**

### Step A1 — Gather a DHCP trace

To capture *all four* DORA messages you must force a full **release + renew**. First find your interface name in Wireshark via **Capture → Options**, then run the de-configure command, **start the capture**, and run the renew command:

```sh
# macOS  (en0 = your interface)
sudo ipconfig set en0 none      # ← de-configure, THEN start Wireshark
sudo ipconfig set en0 dhcp      # ← triggers Discover→Offer→Request→ACK

# Windows
ipconfig /release               # ← give up the address, THEN start Wireshark
ipconfig /renew                 # ← triggers DORA

# Linux  (eth0 = your interface)
sudo ip addr flush dev eth0 && sudo dhclient -r   # ← THEN start Wireshark
sudo dhclient eth0                                # ← triggers DORA
```

Wait a few seconds after the renew, then **stop** the capture.

> Can't capture live (or didn't catch all four)? Use the author's trace **`dhcp-wireshark-trace1-1.pcapng`** from `gaia.cs.umass.edu/wireshark-labs/wireshark-traces-9e.zip`.

<p align="center">
  <!-- ![DORA capture](./img/labA-step1-dora.png) -->
  <em>Fig. 1 — 📸 <code>img/labA-step1-dora.png</code>: the <code>dhcp</code> filter showing Discover, Offer, Request, and ACK.</em>
</p>

### Step A2 — Read the four DORA messages

Type **`dhcp`** (older builds: **`bootp`**) in the filter, then click each message and read its key fields:

| Message | Source IP | Dest IP | Notable fields to read |
|:---|:---|:---|:---|
| **Discover** | `0.0.0.0` *(no address yet!)* | `255.255.255.255` *(broadcast)* | **Transaction ID**; the **Parameter Request List** in Options — the list of settings the client *wants* (subnet mask, router, DNS server, domain…) |
| **Offer** | the DHCP server's IP | client / broadcast | same Transaction ID; the offered address + options |
| **Request** | `0.0.0.0` | `255.255.255.255` | UDP **source port 68 → destination port 67**; same Transaction ID |
| **ACK** | the DHCP server's IP | client | **Your (client) IP Address**, **IP Address Lease Time**, **Router** (gateway), **Domain Name Server** |

Expand the **ACK → Dynamic Host Configuration Protocol → Options** to read the **lease time**, **router (gateway)**, and **DNS server** you were granted — these are the values that let your host reach the rest of the network.

<p align="center">
  <!-- ![ACK options](./img/labA-step2-ack-options.png) -->
  <em>Fig. 2 — 📸 <code>img/labA-step2-ack-options.png</code>: the DHCP <strong>ACK</strong> with the assigned IP, lease time, Router, and DNS Server options expanded.</em>
</p>

---

**Part 2 — DNS: turn names into addresses**

### Step A3 — Explore names with nslookup

Before capturing, get a feel for DNS with **`nslookup`** (built into Windows, macOS, and Linux). It asks a DNS server for a specific **record type**:

```sh
nslookup www.iitb.ac.in        # Type=A  — a name → its IPv4 (and IPv6/AAAA) address
nslookup -type=NS umass.edu    # Type=NS — the domain's authoritative name servers
nslookup 128.119.245.12        # reverse — an IP → its name (gaia.cs.umass.edu)
```

The output names **which DNS server answered** and whether the answer is **authoritative** (from the domain's own server) or **non-authoritative** (served from a cache). *(More on the nslookup output in the [Session 2 UDP lab](../S2/WIRESHARK_GUIDE.md#what-nslookup-is-and-why-it-uses-udp).)*

<p align="center">
  <!-- ![nslookup A and NS](./img/labA-step3-nslookup.png) -->
  <em>Fig. 3 — 📸 <code>img/labA-step3-nslookup.png</code>: an <code>nslookup</code> A query and a <code>-type=NS</code> query.</em>
</p>

### Step A4 — Capture DNS while browsing

1. **Clear your DNS cache** so the lookup actually hits the network (a cached record sends *no* query):
   ```sh
   sudo killall -HUP mDNSResponder        # macOS
   ipconfig /flushdns                     # Windows
   sudo resolvectl flush-caches           # Linux (Ubuntu 22.04+)
   ```
2. Clear your **browser** cache, then **start a capture**.
3. Visit **`http://gaia.cs.umass.edu/kurose_ross/`** and **stop** the capture.
4. Filter **`dns`**. Find the **Standard query** that resolves `gaia.cs.umass.edu` and its matching **Standard query response** (Wireshark pairs them by the same Transaction ID).

<p align="center">
  <!-- ![DNS capture](./img/labA-step4-dns-capture.png) -->
  <em>Fig. 4 — 📸 <code>img/labA-step4-dns-capture.png</code>: the DNS query/response pair generated by loading the page.</em>
</p>

### Step A5 — Inspect the DNS query & response

Select the query, then the response, and confirm:

- Both ride **UDP**; the **query's destination port is 53** and the **response's source port is 53**.
- The **IP the query was sent to** is your **local / default DNS server** (the one DHCP handed you in the ACK).
- Expand **Queries** and **Answers**: the **query** has **1 question, 0 answers**; the **response** echoes that **1 question** plus **one or more answers**.
- Read each answer's **record type** — **A** (IPv4), **AAAA** (IPv6), **CNAME** (alias), or **NS** (name server) — and the address it returns.

<p align="center">
  <!-- ![DNS response answers](./img/labA-step5-dns-response.png) -->
  <em>Fig. 5 — 📸 <code>img/labA-step5-dns-response.png</code>: the DNS response Answers section with the A record for <code>gaia.cs.umass.edu</code>.</em>
</p>

### Lab A Questions

**Try each one first, then click "Show answer".**

**Q1.** Is the **DHCP Discover** sent over UDP or TCP? What is special about its **source** and **destination** IP addresses?

<details>
<summary>💡 Show answer</summary>

**UDP** (server port 67, client port 68). The **source IP is `0.0.0.0`** — the client has *no address yet*, so it can't use a real one. The **destination is `255.255.255.255`** — the limited broadcast address, because the client doesn't know any DHCP server's address and must reach every host on the segment.
</details>

**Q2.** What field ties the **Discover, Offer, Request, and ACK** together as one exchange?

<details>
<summary>💡 Show answer</summary>

The **Transaction ID (xid)** — a random number the client puts in the Discover/Request, which the server copies into the Offer/ACK. It lets the client match replies to its own request even when several clients (or servers) are active at once.
</details>

**Q3.** In the **DHCP Request**, what are the **UDP source and destination ports**? Why are they that way?

<details>
<summary>💡 Show answer</summary>

**Source port 68 (client) → destination port 67 (server).** DHCP fixes these "well-known" ports so the server always listens on 67 and the client on 68 — necessary because the client has no other way to be addressed before it owns an IP. (The Offer/ACK travel the opposite way, 67 → 68.)
</details>

**Q4.** In the **DHCP ACK**, which field carries the **assigned client IP**, and which options give the **lease time** and the **default gateway**?

<details>
<summary>💡 Show answer</summary>

The assigned address is in the **"Your (client) IP Address"** field. The **IP Address Lease Time** option says how long the lease is valid, and the **Router** option is the **default gateway** (first-hop router). The **Domain Name Server** option lists the DNS resolver(s).
</details>

**Q5.** Is the **DNS** query/response over UDP or TCP? What is the query's **destination port** and the response's **source port**?

<details>
<summary>💡 Show answer</summary>

**UDP**, both on port **53** — the query is sent *to* destination port 53, and the response comes *from* source port 53 (the ports swap on the way back). DNS uses UDP for these small, single-shot lookups; it only falls back to TCP for large responses (e.g. zone transfers).
</details>

**Q6.** To what **IP address** is your DNS query sent, and what server is that?

<details>
<summary>💡 Show answer</summary>

To your **local / default DNS server** — the resolver address your host was given in the DHCP **ACK** (Domain Name Server option). Your computer always asks *its* resolver, which then does the recursive/iterative work of contacting root, TLD, and authoritative servers on your behalf.
</details>

**Q7.** How many **"questions"** and **"answers"** are in the DNS **query** versus the **response**?

<details>
<summary>💡 Show answer</summary>

The **query** has **1 question and 0 answers** (you're asking, not telling). The **response** repeats that **1 question** and adds **one or more answers** (e.g. the A record, often an AAAA too, sometimes a CNAME chain). Wireshark shows these counts in the DNS header's *Questions* / *Answer RRs* fields.
</details>

**Q8.** What record type maps a name → **IPv4**? What does a **`-type=NS`** query return, and how does **DNS caching** change a repeated lookup?

<details>
<summary>💡 Show answer</summary>

An **`A`** record maps name → IPv4 (**`AAAA`** → IPv6). A **`-type=NS`** query returns the **authoritative name servers** for a domain (and usually their IPs "for free"). With **caching**, a second lookup of a name you just resolved sends **no DNS query at all** — your host (or the local server) answers from its **resolver cache** until the record's TTL expires, which is why clearing the cache (Step A4) is needed to *see* the query on the wire.
</details>

---

## Next Steps

- Build the **server side** in [Packet Tracer Labs B & C](./PACKET_TRACER_GUIDE.md): secure the router with **SSH** + an **FTP** service (Lab B), then run your own **DHCP & DNS server** so PCs auto-configure and browse by name (Lab C).
- **Homework (from the README):** complete the Kurose & Ross **DNS** Wireshark lab and write a one-paragraph summary of **DNS poisoning/spoofing**.
- Revisit the [Session 2 Wireshark guide](../S2/WIRESHARK_GUIDE.md) and contrast the **TCP** handshake (HTTP) with the **connectionless UDP** you saw in DHCP/DNS here.
