## Wireshark — Lab A: TCP & UDP in Action

> 🌐 **English** | [日本語](./WIRESHARK_GUIDE.ja.md)

> Companion guide to [Session 2 — The Protocol Stack Deep Dive](./README.md).
> Read this **before** doing **Lab A** in the [README](./README.md#hands-on-lab).
> New to Wireshark? Start with the [Session 1 Wireshark guide](../S1/WIRESHARK_GUIDE.md) for installation and the basics.
>
> ⏱️ **~25 min** · **Objective:** capture a **real TCP connection** (the 3-way handshake *and* a bulk file upload), read its sequence/ack numbers, watch congestion control on a graph — then contrast it with a connectionless **UDP** (DNS) exchange.
>
> *Adapted from "Wireshark Lab: TCP v9" and "Wireshark Lab: UDP v9", supplements to* Computer Networking: A Top-Down Approach, 9th ed.*, J.F. Kurose & K.W. Ross.*

---

- [Wireshark — Lab A: TCP \& UDP in Action](#wireshark--lab-a-tcp--udp-in-action)
  - [What is the TCP 3-Way Handshake?](#what-is-the-tcp-3-way-handshake)
  - [Before You Start](#before-you-start)
  - [Part 1 — Capture a TCP Connection (alice.txt upload)](#part-1--capture-a-tcp-connection-alicetxt-upload)
    - [Step 1 — Get the file and open the upload page](#step-1--get-the-file-and-open-the-upload-page)
    - [Step 2 — Capture the upload](#step-2--capture-the-upload)
    - [Step 3 — One POST, many TCP segments](#step-3--one-post-many-tcp-segments)
    - [Step 4 — Filter `tcp` and find the handshake](#step-4--filter-tcp-and-find-the-handshake)
    - [Step 5 — Read the SYN / SYN-ACK / ACK](#step-5--read-the-syn--syn-ack--ack)
    - [Step 6 — Watch congestion control (Stevens graph)](#step-6--watch-congestion-control-stevens-graph)
  - [Part 2 — Capture UDP (nslookup / DNS)](#part-2--capture-udp-nslookup--dns)
    - [What `nslookup` Is and Why It Uses UDP](#what-nslookup-is-and-why-it-uses-udp)
    - [Reading the `nslookup` Output](#reading-the-nslookup-output)
  - [Lab A Questions](#lab-a-questions)
  - [Practice Exercises](#practice-exercises)
  - [Next Steps](#next-steps)

---

### What is the TCP 3-Way Handshake?

**TCP (Transmission Control Protocol)** is **connection-oriented**: before any data is exchanged, the two hosts agree to talk by trading three small control segments. This is the **3-way handshake**, and it uses two of the TCP header **flags** — **`SYN`** (synchronize) and **`ACK`** (acknowledge):

| # | Direction | Flags | Meaning |
|:---:|:---|:---:|:---|
| 1 | You → Server | **`SYN`** | "Can we talk? Here's my starting sequence number." |
| 2 | Server → You | **`SYN, ACK`** | "Yes — here's *my* sequence number, and I acknowledge yours." |
| 3 | You → Server | **`ACK`** | "Confirmed. Let's go." |

After these three packets the connection is **established** and real data can flow — in this lab, a 150 KB file (`alice.txt`) uploaded via an HTTP `POST`. Each side picks an **Initial Sequence Number (ISN)**, and the handshake synchronizes those numbers — that's what `SYN` literally means.

By contrast, **UDP (User Datagram Protocol)** is **connectionless** — there's *no* handshake. A host just fires a datagram (like a DNS query) and hopes for a reply. You'll see both behaviours in this lab.

---

### Before You Start

- [ ] **Wireshark installed** — see the [Session 1 install steps](../S1/WIRESHARK_GUIDE.md#installing-wireshark) if you haven't yet.
- [ ] Review the **TCP/UDP header** fields in the [README Lecture](./README.md#lecture) (ports, sequence/ack numbers, flags, the sliding window).
- [ ] **VPN off**, browser cache cleared (same [pre-flight checklist](../S1/WIRESHARK_GUIDE.md#before-you-capture-pre-flight-checklist) as Session 1).
- [ ] Can't capture live? Download the author's trace **`tcp-wireshark-trace1-1`** from `gaia.cs.umass.edu/wireshark-labs/wireshark-traces-9e.zip` and open it instead.

---

### Part 1 — Capture a TCP Connection (alice.txt upload)

We'll capture a real TCP transfer by **uploading a file** to a web server with HTTP `POST` — a large enough transfer to show the handshake, sequence numbers, and congestion control all in one trace.

#### Step 1 — Get the file and open the upload page

1. In your browser, open `http://gaia.cs.umass.edu/wireshark-labs/alice.txt` and **save it** as a `.txt` file on your computer.
2. Go to `http://gaia.cs.umass.edu/wireshark-labs/TCP-wireshark-file1.html` — the upload page below.
3. Click **Browse…** and select the `alice.txt` you just saved. **Don't press Upload yet.**

<p align="center">
  <img src="./img/TCP%203-Way%20Handshake/fig1-upload-page.png" alt="The alice.txt upload page" width="640"><br>
  <em>Fig. 1 — The upload page. Choose <code>alice.txt</code> with <strong>Browse…</strong>, but don't click Upload until Wireshark is capturing.</em>
</p>

#### Step 2 — Capture the upload

1. **Start Wireshark** capturing on your active interface.
2. Back in the browser, click **"Upload alice.txt file"**. Wait for the short congratulations message.
3. **Stop** the capture. Your trace now contains the whole TCP conversation with `gaia.cs.umass.edu`.

<p align="center">
  <img src="./img/TCP%203-Way%20Handshake/fig2-upload-success.png" alt="Upload complete, with the captured trace in Wireshark" width="700"><br>
  <em>Fig. 2 — Upload complete and the packet trace captured — you're ready to analyse the TCP transfer.</em>
</p>

#### Step 3 — One POST, many TCP segments

Find the **`HTTP POST`** message in the list and expand it. The file is ~152 KB — far too big for a single TCP segment — so the POST is **spread across ~106 TCP segments**. Wireshark notes this for you (and tells you which packet holds the start of the POST).

<p align="center">
  <img src="./img/TCP%203-Way%20Handshake/fig3-http-post-expanded.png" alt="The HTTP POST message expanded, spread over many TCP segments" width="700"><br>
  <em>Fig. 3 — The HTTP <code>POST</code> carrying <code>alice.txt</code> is reassembled from ~106 TCP segments — the byte stream sliced to fit the network.</em>
</p>

#### Step 4 — Filter `tcp` and find the handshake

Type **`tcp`** in the filter bar and press Enter. The listing now shows the **TCP segments** instead of the HTTP view. Near the top you'll see the connection set up: the **SYN**, the **SYN-ACK** from gaia, then the data segments (the first one — carrying the POST — follows the handshake).

<p align="center">
  <img src="./img/TCP%203-Way%20Handshake/fig4-tcp-segments.png" alt="TCP segments with the SYN, SYN-ACK, and first POST segment annotated" width="720"><br>
  <em>Fig. 4 — The TCP segments: the <strong>SYN</strong>, gaia's <strong>SYN-ACK</strong>, and the first data segment carrying the <code>POST</code>. Client <code>192.168.86.68</code> ↔ gaia <code>128.119.245.12 : 80</code>.</em>
</p>

#### Step 5 — Read the SYN / SYN-ACK / ACK

Click each handshake packet and expand **Transmission Control Protocol → Flags** to confirm what makes each one special:

| # | Segment | Flag bits set | What identifies it |
|:---:|:---|:---|:---|
| 1 | **SYN** | `SYN = 1`, `ACK = 0` | the client opening the connection |
| 2 | **SYN-ACK** | `SYN = 1`, `ACK = 1` | gaia agreeing *and* acknowledging your SYN |
| 3 | **ACK** | `SYN = 0`, `ACK = 1` | the client confirming — connection established |

> 💡 The **raw Sequence Number** of the SYN is each side's ISN. The SYN-ACK's **Acknowledgement number = your SYN's sequence number + 1** (the SYN flag "consumes" one sequence number). By default Wireshark shows *relative* numbers starting at 0, so the ACK number usually displays as `1`.

#### Step 6 — Watch congestion control (Stevens graph)

Because the upload is bulk data, you can *see* TCP manage its sending rate over time. Select a client→gaia TCP segment, then **Statistics → TCP Stream Graphs → Time-Sequence-Graph (Stevens)**.

**How to read the axes.** The graph plots **every data segment the client sent** as a dot:

- **X-axis = time** (seconds since the connection started).
- **Y-axis = sequence number** — i.e. the **cumulative number of bytes** handed to the network so far. A dot at `(0.05 s, 12000)` means "by 0.05 s, 12 000 bytes had been sent."

Because the sequence number only ever *grows*, the curve always climbs left-to-right. **The shape of that climb is the story of TCP**, and you read it like this:

| What you see on the graph | What TCP is doing | Why |
|:---|:---|:---|
| A **steep, near-vertical rise** (a stack of dots) | A **"fleet"** — a whole window of segments sent **back-to-back** | The send window let many segments go at once, in a burst. |
| A **flat horizontal gap** after a fleet | The sender is **paused, waiting for ACKs** | It has filled its window; it can't send more until ACKs free up space (this pause ≈ one **round-trip time**). |
| Each fleet **taller than the last** | **Slow start** — the window is *doubling* every RTT | Every ACK lets TCP grow its congestion window exponentially while probing for capacity. |
| Later, fleets grow by a **constant** amount | **Congestion avoidance** — additive (linear) increase | Past the slow-start threshold, TCP adds ~1 segment per RTT to avoid overshooting. |
| The **overall slope** of the curve | The connection's **throughput** | Steeper = more bytes per second. A flattening slope means TCP hit a limit (bandwidth or the receiver's window). |
| A **flat stall then a backward/repeat jump** | A **lost segment + retransmission** | The sender stops, times out (or sees duplicate ACKs), and resends — visible as a break in the smooth climb. |

So the classic **staircase** — *burst up, wait flat, bigger burst up, wait flat* — is TCP's **ACK-clocking** in action: send a window, wait for the ACKs that prove it arrived, then send a larger window. That feedback loop is exactly how TCP discovers how fast it's allowed to go without overwhelming the path.

<p align="center">
  <img src="./img/TCP%203-Way%20Handshake/fig5-stevens-plot.png" alt="Stevens time-sequence plot of the TCP transfer" width="680"><br>
  <em>Fig. 5 — Stevens sequence-number-vs-time plot for <code>192.168.86.68:55639 → 128.119.245.12:80</code>. Each vertical stack is a fleet; each flat gap is a wait-for-ACK. The fleets get taller over time — TCP <strong>slow start</strong> doubling the window each RTT.</em>
</p>

<p align="center">
  <img src="./img/TCP%203-Way%20Handshake/fig6-stevens-altview.png" alt="An alternative view of the same TCP sequence-number data" width="680"><br>
  <em>Fig. 6 — Another view of the same data, making the <strong>periodicity</strong> obvious: one fleet of back-to-back segments per round-trip, repeating. The spacing between bursts is roughly one RTT.</em>
</p>

> 💡 **Try it:** hover or zoom into the first few fleets. Count the segments in each — if fleet 2 has roughly twice as many as fleet 1, you've just *measured* slow start's exponential growth straight off the wire.

---

### Part 2 — Capture UDP (nslookup / DNS)

UDP is the opposite of all that ceremony — **no handshake, no sequence numbers, no ACKs**. The easiest way to generate a UDP exchange is a DNS lookup with **`nslookup`**.

#### What `nslookup` Is and Why It Uses UDP

**`nslookup`** ("name server lookup") is a small command-line tool built into Windows, macOS, and Linux that asks the **Domain Name System (DNS)** to translate a **name** (like `www.nyu.edu`) into an **IP address** — the same lookup your browser does silently before every connection. Running it by hand lets you *see* that resolution step on its own.

It's perfect for this lab because **DNS runs over UDP**: `nslookup` sends a single **query datagram** to a DNS server on **port 53**, and the server sends a single **response datagram** back. No connection is set up or torn down — exactly the connectionless behaviour we want to contrast with TCP.

```sh
# Look up a name (any of these work on the right OS):
nslookup www.nyu.edu          # Windows / macOS / Linux
dig www.nyu.edu               # macOS / Linux alternative (more detail)
```

> 💡 Pick a name you **haven't visited recently** — if the answer is already in your DNS cache, your computer may not send a query at all, and you'll capture nothing.

**Capture it:**

1. **Start a new capture** in Wireshark.
2. Run `nslookup www.nyu.edu` in a terminal.
3. **Stop** the capture and filter with **`udp`** (or `dns`). Select the first segment and expand **User Datagram Protocol** in the details pane.
4. Confirm the UDP header has exactly **four** fields — **Source Port**, **Destination Port**, **Length**, **Checksum** — just 8 bytes. The query's **destination port is 53**; the reply's **source port is 53**, with the ports swapped.

<p align="center">
  <img src="./img/UDP/nslookup.png" alt="Running nslookup, which sends a DNS query over UDP port 53" width="560"><br>
  <em>Fig. 7 — <code>nslookup www.nyu.edu</code> asks the DNS server on port 53 — a single UDP query/response, no connection setup.</em>
</p>

#### Reading the `nslookup` Output

The tool prints **which server it asked** first, then the **answer**:

| Line | Example | What it tells you |
|:---|:---|:---|
| **Server / Address** | `128.119.240.1#53` | The **DNS resolver** you queried, and the port — **`#53`** confirms DNS-over-UDP. |
| **Non-authoritative answer** | — | The answer came from a **cache**, not from the domain's own authoritative name server. |
| **canonical name** | `www.nyu.edu → WEB.GSLB.nyu.edu` | A **`CNAME`** (alias): the name you asked for really points at another name. |
| **Name / Address (IPv4)** | `216.165.47.12` | The **`A` record** — the IPv4 address the name resolves to. |
| **Name / Address (IPv6)** | `2607:f600:1002:6113::100` | The **`AAAA` record** — the IPv6 address for the same name. |

So one tidy `nslookup` maps directly onto what Wireshark shows: a **query** for `www.nyu.edu` going out to `:53`, and a **response** carrying the CNAME, A, and AAAA records back — all inside two UDP datagrams.

> Notice the contrast: the TCP transfer in Part 1 needed a handshake, sequence/ack numbers, and a whole congestion-control graph; this DNS lookup is **one datagram out, one back**. That's the trade-off — TCP earns reliability, UDP buys speed.

---

### Lab A Questions

**Try each one first, then click "Show answer".**

**Q1.** What **IP address and TCP port** does the client use, and what are **gaia's** IP and port for this connection?

<details>
<summary>💡 Show answer</summary>

In this trace the **client** is `192.168.86.68` using a random **ephemeral** high port (e.g. `55639`). **gaia.cs.umass.edu** is `128.119.245.12`, receiving on the well-known port **`80`** (HTTP). On the reply the ports simply swap (gaia:80 → client:55639). Your own numbers will differ, but the *shape* is the same.
</details>

**Q2.** What in a segment identifies it as a **SYN**, and what identifies the **SYN-ACK**?

<details>
<summary>💡 Show answer</summary>

A **SYN** has the **SYN flag set to 1 and ACK to 0**. The **SYN-ACK** has **both SYN = 1 and ACK = 1** — it simultaneously opens gaia's side *and* acknowledges your SYN. The SYN-ACK's **Acknowledgement number = your SYN's sequence number + 1**.
</details>

**Q3.** Why was the HTTP `POST` **spread across ~106 TCP segments** instead of one?

<details>
<summary>💡 Show answer</summary>

`alice.txt` is ~152 KB, far larger than the **Maximum Segment Size** (typically ~1460 bytes). TCP presents the application a single byte *stream*, but it must **slice that stream into segments** that fit the network's MTU — so one big POST becomes ~106 segments, each tagged by its sequence number so the receiver can reassemble them in order.
</details>

**Q4.** On the **Stevens graph**, the early "fleets" of segments grow in size each round trip. What TCP phase is this?

<details>
<summary>💡 Show answer</summary>

**Slow start.** TCP begins conservatively and roughly **doubles** its congestion window every RTT, so each fleet is bigger than the last — the steep, accelerating climb you see early in the plot. It later flattens into **congestion avoidance** (linear growth) once it nears the available bandwidth.
</details>

**Q5.** How many fields are in the **UDP header**, and what are they?

<details>
<summary>💡 Show answer</summary>

**Four**, 8 bytes total: **Source Port**, **Destination Port**, **Length** (header + data), and **Checksum**. That's the entire protocol — no sequence numbers, acknowledgements, or flags, which is exactly why UDP is fast and lightweight.
</details>

**Q6.** What is the **protocol number for UDP** (in the IP header), the **largest possible port number**, and the **maximum UDP payload**?

<details>
<summary>💡 Show answer</summary>

UDP's IP **Protocol number is `17`**. Ports are 16-bit, so the largest is **65,535**. The UDP **Length** field is also 16-bit (max 65,535) and counts the 8-byte header, so the maximum payload is **65,535 − 8 = 65,527 bytes**. (TCP, for comparison, is IP protocol `6`.)
</details>

---

### Practice Exercises

Work through these on your own to lock in TCP and UDP. **Tick each box** as you finish it, and save a screenshot or note of the result.

- [ ] **1. Map the handshake.** In your `alice.txt` capture, find the `SYN → SYN-ACK → ACK`. *Record:* your **client IP + ephemeral port** and **gaia's IP + port 80**, and confirm the flag bits (`SYN=1/ACK=0`, then `SYN=1/ACK=1`).
- [ ] **2. Count the segments.** Locate the HTTP `POST` and read how many TCP segments it spans. *Record:* the segment count, and explain in one line why one POST needs that many (think **MSS**).
- [ ] **3. Read the Stevens graph.** Open **Statistics → TCP Stream Graphs → Time-Sequence (Stevens)**. *Record:* the number of segments in the **first two fleets** — is fleet 2 ≈ 2× fleet 1 (slow start)? Mark roughly where the climb turns from steep to linear (**congestion avoidance**).
- [ ] **4. Measure RTT.** Use **Statistics → TCP Stream Graphs → Round-Trip-Time Graph** (or subtract the POST's send time from its ACK time). *Record:* the **RTT of the first data segment**.
- [ ] **5. Capture UDP.** Run `nslookup` for a fresh hostname while capturing, then filter `udp`. *Record:* the **four UDP header fields**, the **destination port (53)**, and the **A / AAAA** addresses returned.
- [ ] **Stretch — Watch the teardown.** Apply `tcp.flags.fin == 1` and find the `FIN/ACK` packets that close the connection. *Record:* how many packets it takes to tear down vs. the 3 to set up.

> [!TIP]
> Treat each *Record* line as your deliverable — together these prove you can read a handshake, follow segmentation, interpret a congestion graph, and tell TCP from UDP on sight.

---

### Next Steps

- **Capture a teardown:** apply `tcp.flags.fin == 1` and find the `FIN/ACK` packets that *close* the connection — the mirror image of the handshake.
- **Plot the RTT:** select a client→gaia segment, then **Statistics → TCP Stream Graphs → Round Trip Time Graph**, to see how latency varied across the transfer.
- **Peek at HTTPS:** load an `https://` site and note the TCP handshake is still visible, but the payload is encrypted (TLS).
- Move on to the [Packet Tracer guide](./PACKET_TRACER_GUIDE.md) for Lab B, where you'll design and build a multi-subnet network from scratch.
