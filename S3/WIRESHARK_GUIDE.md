## Wireshark — Labs A & C: Network Services & Security Analysis

> Companion guide to [Session 3 — Network Services & Security](./README.md).
> Read this **before** doing **Lab A** and **Lab C** in the [README](./README.md#️-hands-on-lab).
> New to Wireshark? Start with the [Session 1 Wireshark guide](../S1/WIRESHARK_GUIDE.md) for installation and the basics, and the [Session 2 guide](../S2/WIRESHARK_GUIDE.md) for filters & Follow-Stream.

---

- [Wireshark — Labs A \& C: Network Services \& Security Analysis](#wireshark--labs-a--c-network-services--security-analysis)
  - [🧪 Lab A — DHCP & DNS Analysis](#-lab-a--dhcp--dns-analysis)
    - [🔑 Background: DORA & name resolution](#-background-dora--name-resolution)
    - [Step A1 — Start capturing](#step-a1--start-capturing)
    - [Step A2 — Release & renew your IP](#step-a2--release--renew-your-ip)
    - [Step A3 — Find the 4 DORA packets](#step-a3--find-the-4-dora-packets)
    - [Step A4 — Inspect a DNS query](#step-a4--inspect-a-dns-query)
    - [Step A5 — Inspect the DNS response](#step-a5--inspect-the-dns-response)
    - [📝 Lab A Questions](#-lab-a-questions)
  - [🧪 Lab C — Network Security Analysis](#-lab-c--network-security-analysis)
    - [Step C1 — Spot an ARP anomaly](#step-c1--spot-an-arp-anomaly)
    - [Step C2 — Identify a port scan](#step-c2--identify-a-port-scan)
    - [Step C3 — Plaintext vs. encrypted](#step-c3--plaintext-vs-encrypted)
    - [Step C4 — Reflect: which defense stops which attack?](#step-c4--reflect-which-defense-stops-which-attack)
    - [📝 Lab C Questions](#-lab-c-questions)
  - [➡️ Next Steps](#️-next-steps)

---

## 🧪 Lab A — DHCP & DNS Analysis

**⏱️ ~20 min · Objective:** watch a device **get an address (DHCP)** and **resolve a name (DNS)** — two services that make "the network just work."

### 🔑 Background: DORA & name resolution

When a device joins a network it has no IP yet, so it asks for one. **DHCP** (Dynamic Host Configuration Protocol) answers in a four-step exchange remembered as **DORA**:

| # | Message | From → To | Purpose |
|:---:|:---|:---|:---|
| 1 | **D**iscover | Client → broadcast | "Is there a DHCP server out there?" |
| 2 | **O**ffer | Server → client | "Yes — here's an address you can have." |
| 3 | **R**equest | Client → broadcast | "I'll take that offered address." |
| 4 | **A**ck | Server → client | "It's yours — here's your lease, gateway, and DNS server." |

DHCP runs over **UDP** — server port **67**, client port **68**. Once addressed, the device uses **DNS** (Domain Name System, **UDP port 53**) to turn names like `google.com` into IP addresses via a **query → response** pair.

### Step A1 — Start capturing

Open Wireshark and double-click your active interface to begin a live capture, then leave it running.

<p align="center">
  <!-- ![Start capture](./img/labA-step1-capture.png) -->
  <em>Fig. 1 — 📸 <code>img/labA-step1-capture.png</code>: live capture started on the active interface.</em>
</p>

### Step A2 — Release & renew your IP

Force a fresh DHCP exchange from a terminal:

```sh
# Windows
ipconfig /release
ipconfig /renew

# macOS / Linux
sudo dhclient -r && sudo dhclient
# (macOS alt: sudo ipconfig set en0 NONE && sudo ipconfig set en0 DHCP)
```

<p align="center">
  <!-- ![Release renew](./img/labA-step2-renew.png) -->
  <em>Fig. 2 — 📸 <code>img/labA-step2-renew.png</code>: the release/renew commands in a terminal.</em>
</p>

### Step A3 — Find the 4 DORA packets

1. In the filter bar type **`dhcp`** (older builds: **`bootp`**) and press **Enter**.
2. Identify the four messages: **Discover → Offer → Request → ACK**.
3. Click the **ACK** and expand **Dynamic Host Configuration Protocol → Options** to read the **lease time**, **router (gateway)**, and **DNS server** handed to you.

<p align="center">
  <!-- ![DORA packets](./img/labA-step3-dora.png) -->
  <em>Fig. 3 — 📸 <code>img/labA-step3-dora.png</code>: the four DORA packets, with the ACK's Option fields expanded.</em>
</p>

### Step A4 — Inspect a DNS query

1. Clear the filter, type **`dns`**, press **Enter**.
2. Find a **Standard query** for a name such as `google.com` (browse to it if needed to generate one).
3. Expand **Domain Name System (query) → Queries** to see the **Name** and **Type** requested.

<p align="center">
  <!-- ![DNS query](./img/labA-step4-dnsquery.png) -->
  <em>Fig. 4 — 📸 <code>img/labA-step4-dnsquery.png</code>: a DNS standard query for <code>google.com</code>.</em>
</p>

### Step A5 — Inspect the DNS response

Click the matching **Standard query response** and expand **Answers**. Note the **record type** (e.g. **A** = IPv4, **AAAA** = IPv6, **CNAME** = alias) and the **IP address(es)** returned.

<p align="center">
  <!-- ![DNS response](./img/labA-step5-dnsresponse.png) -->
  <em>Fig. 5 — 📸 <code>img/labA-step5-dnsresponse.png</code>: the DNS response Answers section with the A record.</em>
</p>

### 📝 Lab A Questions

**Try each one first, then click "Show answer".**

**Q1.** Name the **four DHCP messages** in order. Which ones are **broadcast** and why?

<details>
<summary>💡 Show answer</summary>

**Discover → Offer → Request → ACK** (DORA). The client's **Discover** and **Request** are **broadcast** because the client has *no IP yet* and doesn't know the server's address — it must shout to everyone. (The server's Offer/ACK can be unicast or broadcast depending on the client's flags.)
</details>

**Q2.** What **transport protocol and ports** does DHCP use?

<details>
<summary>💡 Show answer</summary>

**UDP**, with the **server on port 67** and the **client on port 68**. (UDP because the client has no IP/connection state yet — a lightweight broadcast fits better than setting up TCP.)
</details>

**Q3.** In your DHCP **ACK**, which option fields told the client how to reach the rest of the network?

<details>
<summary>💡 Show answer</summary>

The **Router (default gateway)** option and the **Domain Name Server** option — plus the **IP Address Lease Time**. Without the gateway the client couldn't leave its subnet; without the DNS server it couldn't resolve names.
</details>

**Q4.** In the DNS response, which **record type** maps a hostname to an **IPv4** address? What about IPv6?

<details>
<summary>💡 Show answer</summary>

An **`A`** record maps a name → **IPv4** address. An **`AAAA`** ("quad-A") record maps a name → **IPv6** address. A **`CNAME`** is an alias pointing one name at another name.
</details>

---

## 🧪 Lab C — Network Security Analysis

**⏱️ ~25 min · Objective:** recognise the *signatures* of common attacks (ARP spoofing, port scans) in a capture, and see why **HTTPS** matters.

> ⚠️ **Authorized use only.** Run scans/spoofing **only on your own lab network or a host you control**. Scanning or spoofing networks you don't own may be illegal.

### Step C1 — Spot an ARP anomaly

1. Filter **`arp`**. Note your normal **request/reply** pairs and the gateway's MAC.
2. Look for the spoofing signature: **two different MACs claiming the same IP**, or a **flood of gratuitous ARP** replies.
3. Use **Analyze → Expert Information** — Wireshark flags **"duplicate IP address detected"** when an IP maps to more than one MAC.

<p align="center">
  <!-- ![ARP anomaly](./img/labC-step1-arp.png) -->
  <em>Fig. 6 — 📸 <code>img/labC-step1-arp.png</code>: two MACs claiming one IP (the ARP-spoofing signature).</em>
</p>

### Step C2 — Identify a port scan

1. From another machine you control, run a SYN scan against a lab host:
   ```sh
   nmap -sS <target-ip>
   ```
2. In Wireshark, filter for bare SYNs:
   ```text
   tcp.flags.syn == 1 && tcp.flags.ack == 0
   ```
3. Observe **many SYNs to sequential ports** from one source — the classic scan pattern. Compare replies: a **SYN-ACK = open** port, a **RST = closed** port.

<p align="center">
  <!-- ![Port scan](./img/labC-step2-scan.png) -->
  <em>Fig. 7 — 📸 <code>img/labC-step2-scan.png</code>: a burst of SYNs to sequential ports from one host.</em>
</p>

### Step C3 — Plaintext vs. encrypted

1. While capturing, visit an **`http://`** site and an **`https://`** site.
2. Filter **`http`** → use **Follow → TCP Stream** and confirm you can **read the content in plaintext**.
3. Filter **`tls`** → confirm the payload is **encrypted** (only the handshake and the SNI server name are visible). *This is why HTTPS matters.*

<p align="center">
  <!-- ![Plaintext vs encrypted](./img/labC-step3-tls.png) -->
  <em>Fig. 8 — 📸 <code>img/labC-step3-tls.png</code>: readable HTTP stream vs. an opaque encrypted TLS payload.</em>
</p>

### Step C4 — Reflect: which defense stops which attack?

Match each defense to the attack it neutralises:

| Attack (signature you found) | Defense that stops it |
|:---|:---|
| ARP spoofing / duplicate-IP | **Dynamic ARP Inspection (DAI)** |
| Rogue DHCP server | **DHCP Snooping** |
| Plaintext credential capture | **HTTPS / TLS** (and SSH over Telnet) |
| DNS spoofing / cache poisoning | **DNSSEC** |

### 📝 Lab C Questions

**Try each one first, then click "Show answer".**

**Q1.** What exactly in the `arp` capture reveals **ARP spoofing**?

<details>
<summary>💡 Show answer</summary>

The same **IP address mapped to two different MAC addresses** (often the attacker rapidly sending **gratuitous ARP** replies to overwrite the gateway's entry in victims' ARP caches). Wireshark's Expert Info calls this **"duplicate IP address detected."** ARP has no authentication, so anyone on the LAN can claim any IP.
</details>

**Q2.** Under the filter `tcp.flags.syn == 1 && tcp.flags.ack == 0`, how do you tell which ports are **open**?

<details>
<summary>💡 Show answer</summary>

You're seeing the scanner's outbound **SYNs**. Look at the **replies**: an **open** port answers with a **SYN-ACK** (and the scanner usually sends RST to avoid completing the handshake — a "half-open"/stealth scan); a **closed** port replies with **RST**. No reply at all often means a firewall is **filtering** it.
</details>

**Q3.** Under `tls`, what can you **still see**, and what is **hidden**?

<details>
<summary>💡 Show answer</summary>

You can still see **metadata**: the IP addresses, ports, the TLS handshake, and often the **SNI** (the server hostname being requested). What's **hidden** is the **application payload** — URLs, headers, credentials, page content are all encrypted. So HTTPS protects the *content*, not the fact that you talked to a given server.
</details>

**Q4.** A rogue DHCP server handed a victim a malicious gateway. Which **switch defense** would have stopped it, and how?

<details>
<summary>💡 Show answer</summary>

**DHCP Snooping.** It lets the switch mark only specific ports as **trusted** to send DHCP Offers/ACKs (the ports leading to the real server). Offers arriving on **untrusted** ports — where a rogue server sits — are **dropped**, so victims never accept the bad gateway/DNS.
</details>

---

## ➡️ Next Steps

- Build the matching **server side** in [Packet Tracer Lab B](./PACKET_TRACER_GUIDE.md) — configure a DHCP pool + DNS records and watch DORA happen in Simulation Mode.
- **Homework (from the README):** complete the Kurose & Ross **DNS** Wireshark lab and write a one-paragraph summary of **DNS poisoning/spoofing**.
- Revisit the [Session 2 Wireshark guide](../S2/WIRESHARK_GUIDE.md) and contrast the **TCP** handshake (HTTP) with the **connectionless UDP** you saw in DHCP/DNS here.
