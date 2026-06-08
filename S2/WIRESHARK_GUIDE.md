## Wireshark — Lab A: The TCP 3-Way Handshake

> Companion guide to [Session 2 — The Protocol Stack Deep Dive](./README.md).
> Read this **before** doing **Lab A** in the [README](./README.md#hands-on-lab).
> New to Wireshark? Start with the [Session 1 Wireshark guide](../S1/WIRESHARK_GUIDE.md) for installation and the basics.
>
> ⏱️ **~20 min** · **Objective:** see a real `SYN → SYN-ACK → ACK` handshake and contrast connection-oriented **TCP** with connectionless **UDP**.

---

- [Wireshark — Lab A: The TCP 3-Way Handshake](#wireshark--lab-a-the-tcp-3-way-handshake)
  - [What is the TCP 3-Way Handshake?](#what-is-the-tcp-3-way-handshake)
  - [Before You Start](#before-you-start)
  - [Lab A — Capture & Read a Handshake](#lab-a--capture--read-a-handshake)
    - [Step A1 — Start a capture](#step-a1--start-a-capture)
    - [Step A2 — Generate traffic](#step-a2--generate-traffic)
    - [Step A3 — Filter for the handshake](#step-a3--filter-for-the-handshake)
    - [Step A4 — Identify SYN → SYN-ACK → ACK](#step-a4--identify-syn--syn-ack--ack)
    - [Step A5 — Follow the TCP stream](#step-a5--follow-the-tcp-stream)
    - [Step A6 — Compare with UDP (DNS)](#step-a6--compare-with-udp-dns)
  - [Lab A Questions](#lab-a-questions)
  - [Next Steps](#next-steps)

---

### What is the TCP 3-Way Handshake?

**TCP (Transmission Control Protocol)** is **connection-oriented**: before any data is exchanged, the two hosts agree to talk by trading three small control segments. This is the **3-way handshake**, and it uses two of the TCP header **flags** — **`SYN`** (synchronize) and **`ACK`** (acknowledge):

| # | Direction | Flags | Meaning |
|:---:|:---|:---:|:---|
| 1 | You → Server | **`SYN`** | "Can we talk? Here's my starting sequence number." |
| 2 | Server → You | **`SYN, ACK`** | "Yes — here's *my* sequence number, and I acknowledge yours." |
| 3 | You → Server | **`ACK`** | "Confirmed. Let's go." |

After these three packets the connection is **established** and real data (e.g. an HTTP `GET`) can flow. Each side also picks an **Initial Sequence Number (ISN)**, and the handshake is how they synchronize those numbers — that's what the `SYN` flag literally means.

By contrast, **UDP (User Datagram Protocol)** is **connectionless** — there's *no* handshake at all. A host just fires off a datagram (like a DNS query) and hopes for a reply. You'll see both behaviours side-by-side in this lab.

---

### Before You Start

- [ ] **Wireshark installed** — see the [Session 1 install steps](../S1/WIRESHARK_GUIDE.md#installing-wireshark) if you haven't yet.
- [ ] Review the **TCP/UDP header** fields in the [README Lecture](./README.md#lecture) (ports, sequence/ack numbers, flags).
- [ ] Use a **plain `http://`** site (e.g. `http://example.com`) so the handshake is easy to spot and not buried in TLS.
- [ ] **VPN off**, browser cache cleared (same [pre-flight checklist](../S1/WIRESHARK_GUIDE.md#before-you-capture-pre-flight-checklist) as Session 1).

---

### Lab A — Capture & Read a Handshake

#### Step A1 — Start a capture

1. Open Wireshark.
2. Double-click your active interface (the one with a moving traffic sparkline — usually `Wi-Fi` or `Ethernet`).
3. Capture begins immediately.

<p align="center">
  <!-- ![Wireshark interface selection](./img/labA-step1-interfaces.png) -->
  <em>Fig. 1 — 📸 <code>img/labA-step1-interfaces.png</code>: the interface list with the live traffic sparkline.</em>
</p>

#### Step A2 — Generate traffic

1. Leave Wireshark running.
2. In a browser, visit a **plain HTTP** site (e.g. `http://example.com`) so the handshake is easy to spot.
3. Return to Wireshark and click the red **🟥 Stop** button after the page loads.

<p align="center">
  <!-- ![Generating traffic](./img/labA-step2-traffic.png) -->
  <em>Fig. 2 — 📸 <code>img/labA-step2-traffic.png</code>: the loaded page + Wireshark still capturing.</em>
</p>

#### Step A3 — Filter for the handshake

1. In the filter bar type:
   ```text
   tcp.flags.syn == 1
   ```
2. Press **Enter**. Now only packets with the **SYN flag set** are shown — these are the start of every TCP connection in your capture.

<p align="center">
  <!-- ![SYN filter applied](./img/labA-step3-synfilter.png) -->
  <em>Fig. 3 — 📸 <code>img/labA-step3-synfilter.png</code>: the <code>tcp.flags.syn == 1</code> filter applied, SYN rows highlighted.</em>
</p>

#### Step A4 — Identify SYN → SYN-ACK → ACK

Look for three consecutive packets between your PC and the server. Click each one and expand **Transmission Control Protocol → Flags** in the detail pane to confirm which flags are set:

| # | Direction | Flags | Meaning |
|:---:|:---|:---:|:---|
| 1 | You → Server | `SYN` | "Can we talk?" |
| 2 | Server → You | `SYN, ACK` | "Yes — and you?" |
| 3 | You → Server | `ACK` | "Confirmed." |

<p align="center">
  <!-- ![3-way handshake](./img/labA-step4-handshake.png) -->
  <em>Fig. 4 — 📸 <code>img/labA-step4-handshake.png</code>: the three handshake packets selected, Flags subtree expanded.</em>
</p>

#### Step A5 — Follow the TCP stream

1. Right-click any packet in the connection → **Follow → TCP Stream**.
2. Observe the full request/response conversation (**red** = your request, **blue** = server reply). Wireshark also auto-builds the filter `tcp.stream eq N` so you see *only* this one connection.

<p align="center">
  <!-- ![Follow TCP stream](./img/labA-step5-stream.png) -->
  <em>Fig. 5 — 📸 <code>img/labA-step5-stream.png</code>: the Follow TCP Stream window.</em>
</p>

#### Step A6 — Compare with UDP (DNS)

1. Close the stream window, clear the filter, and apply:
   ```text
   udp.port == 53
   ```
2. Notice: **no handshake** — just a query → response pair. This is connectionless **UDP** in action.

<p align="center">
  <!-- ![DNS over UDP](./img/labA-step6-dns.png) -->
  <em>Fig. 6 — 📸 <code>img/labA-step6-dns.png</code>: a DNS query/response pair under the <code>udp.port == 53</code> filter.</em>
</p>

---

### Lab A Questions

**Try each one first, then click "Show answer".**

**Q1.** In your captured handshake, what are the **source and destination ports**? Which one is the well-known server port?

<details>
<summary>💡 Show answer</summary>

The **destination port** of the `SYN` is the **server's well-known port** — `80` for HTTP (or `443` for HTTPS). Your computer's **source port** is a random high number (an *ephemeral* port, e.g. `54321`) that the OS picks for this connection. In the `SYN-ACK` the ports are swapped (server → you).
</details>

**Q2.** Why does TCP need **three** messages? Why not just two (`SYN` then `ACK`)?

<details>
<summary>💡 Show answer</summary>

Because **both sides** must (a) announce their own Initial Sequence Number and (b) acknowledge the other's. Two messages would only synchronize one direction. The three-step exchange confirms that **each host can both send and receive** before any data flows — packet 2 (`SYN,ACK`) cleverly combines the server's `SYN` with its acknowledgement of yours, which is why it's three and not four.
</details>

**Q3.** In **Follow TCP Stream**, what do the **red** and **blue** colours represent?

<details>
<summary>💡 Show answer</summary>

**Red** is traffic from the **client (you)** to the server — e.g. your HTTP `GET` request. **Blue** is the **server's** reply back to you — e.g. the `200 OK` and the page content. It reassembles the raw bytes of the whole conversation in order, so you read it like a transcript.
</details>

**Q4.** The DNS exchange over UDP had **no handshake**. Why doesn't DNS need one?

<details>
<summary>💡 Show answer</summary>

DNS queries are tiny, single-shot request/response transactions where **speed matters more than guaranteed delivery**. Setting up and tearing down a TCP connection (3 packets each way) would add pointless overhead for one small question. If a UDP DNS reply is lost, the resolver simply **asks again**. That trade-off — less reliability for less overhead — is exactly why UDP exists alongside TCP.
</details>

**Q5.** *(Stretch)* Look at the **Sequence number** of the `SYN` and the **Acknowledgment number** of the `SYN-ACK`. What's the relationship?

<details>
<summary>💡 Show answer</summary>

The server's **Acknowledgment number = your SYN's Sequence number + 1**. The `SYN` flag "consumes" one sequence number, so acknowledging it means "I received up to your ISN; I now expect byte ISN+1." (Wireshark shows *relative* sequence numbers starting at 0 by default, so you'll typically see the ACK number as `1`.)
</details>

---

### Next Steps

- **Capture a teardown too:** apply `tcp.flags.fin == 1` and find the `FIN/ACK` packets that *close* a connection — the mirror image of the handshake.
- **Peek at HTTPS:** load an `https://` site and note that the TCP handshake is still visible, but the payload is encrypted (TLS) — you'll cover this in a later session.
- Move on to the [Packet Tracer guide](./PACKET_TRACER_GUIDE.md) for Labs B & C, where you'll watch this encapsulation happen hop-by-hop and design a multi-subnet network.
