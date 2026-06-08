## Wireshark — Getting Started & Lab 1

> Companion guide to [Session 1 — Introduction to Network Analysis & The OSI Model](./README.md).
> Read this **before** doing **Lab A — Wireshark: First Capture** in the [README](./README.md#hands-on-lab).
>
> *Adapted from "Wireshark Lab: Getting Started v9.0", a supplement to* Computer Networking: A Top-Down Approach, 9th ed.*, J.F. Kurose and K.W. Ross.*

---

- [Wireshark — Getting Started \& Lab 1](#wireshark--getting-started--lab-1)
  - [🦈 What is Wireshark?](#-what-is-wireshark)
  - [🧩 How a Packet Sniffer Works](#-how-a-packet-sniffer-works)
  - [⬇️ Installing Wireshark](#️-installing-wireshark)
  - [✅ Before You Capture: Pre-flight Checklist](#-before-you-capture-pre-flight-checklist)
  - [🖥️ The Wireshark Start Screen](#️-the-wireshark-start-screen)
  - [🪟 The Five Parts of the Wireshark Window](#-the-five-parts-of-the-wireshark-window)
  - [🌐 A Quick Word on HTTP (the traffic we'll capture)](#-a-quick-word-on-http-the-traffic-well-capture)
  - [🧪 Lab 1 — Taking Wireshark for a Test Run](#-lab-1--taking-wireshark-for-a-test-run)
  - [📝 Lab 1 Questions](#-lab-1-questions)
  - [➡️ Next Steps](#️-next-steps)

---

### 🦈 What is Wireshark?

**Wireshark** is the world's most popular **packet sniffer** (also called a *packet analyzer* or *protocol analyzer*). It is a free, open-source tool that lets you *see* the network protocols running on your computer "in action" — observing the actual sequence of messages exchanged between protocol entities, drilling down into the details of each protocol field, and watching how your actions (like loading a web page) cause protocols to send and receive messages.

> *"Tell me and I forget. Show me and I remember. Involve me and I understand."* — Chinese proverb

A packet sniffer is **passive**. It only *observes* messages sent and received by applications and protocols running on your computer — it never sends packets of its own, and packets are never explicitly addressed to it. Instead, it receives a **copy** of every frame that passes through your network interface.

---

### 🧩 How a Packet Sniffer Works

<p align="center">
  <img src="./img/packet%20sniffer%20structure.png" alt="Packet sniffer structure" width="600"><br>
  <em>Fig. 1 — Structure of a packet sniffer.</em>
</p>

A packet sniffer (the dashed rectangle above) is an addition to the normal networking software on your computer, and it has **two parts**:

1. **Packet capture library (`pcap`)** — receives a *copy* of every **link-layer frame** sent or received over a given interface (Ethernet or Wi-Fi). Because *all* higher-layer messages — HTTP, DNS, TCP, UDP, IP — are eventually encapsulated into link-layer frames, capturing every frame gives you **every message** sent/received by **every** protocol and application on your machine.
2. **Packet analyzer** — understands the structure of the captured frames and the protocols nested inside them. It decodes and displays the contents of each protocol field (Ethernet → IP → TCP/UDP → HTTP, etc.) in a readable form.

This maps directly onto the **OSI / TCP-IP encapsulation** you learned in the [Session 1 lecture](./README.md#-lecture): the capture library grabs the raw **Layer 2 frame**, and the analyzer peels back **Layer 3 (IP)**, **Layer 4 (TCP/UDP)**, and **Layer 7 (application)** for you.

---

### ⬇️ Installing Wireshark

Download Wireshark for your operating system from the official site:

| Platform | How to install |
| :--- | :--- |
| **Windows** | Download the installer from [wireshark.org/download.html](https://www.wireshark.org/download.html). Accept installing **Npcap** when prompted (this is the capture driver). |
| **macOS** | Download the `.dmg` from the site, **or** run `brew install --cask wireshark`. Install **ChmodBPF** when prompted so capturing works without `sudo`. |
| **Linux** | `sudo apt install wireshark` (Debian/Ubuntu) or `sudo dnf install wireshark` (Fedora). Answer **Yes** to "allow non-superusers to capture packets", then add yourself: `sudo usermod -aG wireshark $USER` and log back in. |

There is also a short official intro video: <https://youtu.be/kCwd2YoJcvg>

---

### ✅ Before You Capture: Pre-flight Checklist

These settings make sure your traffic is **readable plaintext** instead of encrypted noise:

- [ ] **Turn off any VPN.** A VPN encrypts upper-layer (HTTP/TCP) information, hiding it from Wireshark.
- [ ] **Disable HTTP/3 and QUIC in your browser.** As of 2025 most browsers use these by default, and they encrypt upper-layer information. (Guide: <https://techysnoop.com/disable-quic-protocol-in-chrome-edge-firefox/>)
- [ ] **Use `http://` URLs, not `https://`.** HTTPS encrypts the frame contents.
- [ ] **Turn off browser privacy/anti-tracking features** that might block or reroute traffic.
- [ ] **Clear your browser cache and history** so the page is actually fetched over the network (not served from cache).

---

### 🖥️ The Wireshark Start Screen

When you launch Wireshark, you'll see a startup screen like the one below. **Don't panic if yours looks a little different** — Wireshark runs on many platforms and toolkits, but the functionality is the same.

<p align="center">
  <img src="./img/Initial%20Wireshark%20Screen.png" alt="Initial Wireshark screen" width="620"><br>
  <em>Fig. 2 — The initial Wireshark screen.</em>
</p>

The key area is the **Capture** section, which lists your network **interfaces**. The Mac in this screenshot has its **`Wi-Fi: en0`** interface highlighted in blue — every packet to/from this computer passes through it, so that's where we capture.

> 💡 **Which interface is mine?** The active interface usually shows a **moving line-graph (sparkline)** of live traffic next to its name. **Double-click** it to start capturing immediately. On other machines, pick whichever interface gives you Internet connectivity (typically Wi-Fi or Ethernet).

You can also start a capture via the **Capture → Options** menu (Mac) or **Capture → Interfaces** (Windows), then select the interface and click **Start**.

---

### 🪟 The Five Parts of the Wireshark Window

Once capturing, the window looks like Fig. 3. It has **five major components**:

<p align="center">
  <img src="./img/Wireshark%20window%2C%20during%20and%20after%20capture.png" alt="Wireshark window during and after capture" width="680"><br>
  <em>Fig. 3 — The Wireshark window, during and after a capture.</em>
</p>

1. **Command menus** (top) — standard pulldown menus. The two you'll use most now are **File** (save/open capture files, exit) and **Capture** (start/stop capturing).
2. **Display filter field** — type a protocol name or expression here to show only the packets you care about (e.g. `http`). Filtering is covered in depth in the [README's Display Filters deep-dive](./README.md#-deep-dive-wireshark-display-filters).
3. **Packet-listing window** — one line per packet: the Wireshark packet **number**, **time**, **source** and **destination IP**, highest-level **protocol**, and a short **info** summary. Click any column header to sort.
4. **Packet-header details window** — a tree of the selected packet's protocol layers. Expand/collapse each layer (Frame → Ethernet → IP → TCP/UDP → application) with the ▸/▾ triangles. This is where you *see encapsulation* layer by layer.
5. **Packet-contents window** — the entire raw frame in **hexadecimal and ASCII**.

> Components 3, 4 and 5 are linked: select a packet in the listing (3), and its layers appear in the details (4) and its raw bytes in the contents (5).

---

### 🌐 A Quick Word on HTTP (the traffic we'll capture)

In the lab below you'll load a web page, so the traffic you capture will be **HTTP** — the protocol web browsers and servers use. At its core, HTTP is a simple **request → reply** conversation between a browser (the *client*) and a web server:

<p align="center">
  <img src="./img/Basic%20Application%20Logic%20to%20Get%20a%20Web%20Page.png" alt="A browser asking a server for a web page" width="560"><br>
  <em>Fig. 4 — The browser ("Bob") asks the server ("Larry") for a page; the server sends the file back.</em>
</p>

Looking a little closer, each message carries an **HTTP header** describing it. The browser sends a **`GET`** request naming the file it wants; the server replies with a status (**`200 OK`**) header followed by the file's data, which may span several messages:

<p align="center">
  <img src="./img/HTTP%20GET%20Request%2C%20HTTP%20Reply%2C%20and%20One%20Data-Only%20Message.png" alt="HTTP GET request, HTTP reply, and a data-only message" width="560"><br>
  <em>Fig. 5 — An HTTP <code>GET</code> request, the <code>200 OK</code> reply, and a follow-on data message.</em>
</p>

These are exactly the messages you'll hunt for in Wireshark: the **`GET`** your browser sends, and the **`200 OK`** the server sends back. Keep this picture in mind as you do the capture.

---

### 🧪 Lab 1 — Taking Wireshark for a Test Run

The best way to learn the tool is to use it. This lab captures a real HTTP page download and finds the `GET` request inside it.

1. **Start your web browser** (incognito/private mode recommended).
2. **Start Wireshark.** You'll see the start screen (Fig. 2) — no packets are being captured yet.
3. **Begin capturing.** Double-click your active interface (e.g. `Wi-Fi: en0`), or use **Capture → Options → Start**. The window now fills with live packets (Fig. 3).
4. **Generate traffic.** In your browser, enter this URL and load it:
   ```text
   http://gaia.cs.umass.edu/wireshark-labs/INTRO-wireshark-file1.html
   ```
   Your browser contacts the HTTP server at `gaia.cs.umass.edu`, exchanges HTTP messages, and downloads a one-line "congratulations" page. Those frames (and lots of others) are now captured.
5. **Stop the capture.** Click the red square **🟥 Stop** button, or use **Capture → Stop**. Notice how many *other* protocols showed up (DNS, TCP, TLS, QUIC…) even though you only loaded one page — there's always more going on than meets the eye.
6. **Apply a display filter.** In the filter field type `http` (lower-case — *all* protocol names in Wireshark are lower-case) and press **Enter / Apply**. Only HTTP messages remain visible. A **green** filter bar means the syntax is valid:

   <p align="center">
     <img src="./img/Screenshot%202026-06-08%20at%209.41.54.png" alt="The http display filter typed into the green filter bar" width="680"><br>
     <em>Fig. 6 — The <code>http</code> display filter in the green (valid) filter bar.</em>
   </p>
7. **Find the HTTP `GET`.** In the packet listing, look for the line whose Info shows **`GET`** followed by the `gaia.cs.umass.edu` URL. Click it.
8. **Explore the layers (OSI in action).** In the **packet-header details** pane, collapse the **Frame**, **Ethernet II**, **Internet Protocol**, and **Transmission Control Protocol** sections (▸), and **expand the Hypertext Transfer Protocol** section (▾). You can now read the raw HTTP `GET` request — its `Host`, `User-Agent`, and `Accept` headers. Your screen should resemble Fig. 3 with HTTP maximized.
9. **Exit Wireshark.** 🎉 **Congratulations — you've completed your first Wireshark lab!**

> 💾 To keep your capture for later, use **File → Export Specified Packets…** (or **File → Save As…**) and save it as a `.pcap` / `.pcapng` file — e.g. `my_first_capture.pcap`.

---

### 📝 Lab 1 Questions

Answer these from your own capture (or from a downloaded trace file, if you couldn't capture live). **Try each one first, then click "Show answer".**

**Q1.** **Which of these protocols appear** in the *Protocol* column of your trace: TCP, QUIC, HTTP, DNS, UDP, TLSv1.2?

<details>
<summary>💡 Show answer</summary>

All six can show up. Loading the page itself uses **DNS** (to resolve `gaia.cs.umass.edu`), then **TCP** + **HTTP** (to fetch it). The rest — **UDP**, **QUIC**, and **TLSv1.2** — come from *background* traffic other apps on your computer generate while you capture. (Exactly which appear depends on your machine; the essential three for this page are **DNS, TCP, HTTP**.)
</details>

**Q2.** **How long** did it take from when the HTTP `GET` was sent until the HTTP `200 OK` reply was received? *(The Time column defaults to seconds since the trace began. To switch to clock time: **View → Time Display Format → Time-of-day**.)*

<details>
<summary>💡 Show answer</summary>

Subtract the **Time** value of the `GET` packet from the **Time** value of the `200 OK` packet. This is the round-trip time to the server and is typically a small fraction of a second (often **~0.02–0.3 s**, depending on your distance to the server and network conditions). Example: OK at `1.452 s` − GET at `1.410 s` = **0.042 s (42 ms)**.
</details>

**Q3.** **What is the IP address of `gaia.cs.umass.edu`** (a.k.a. `www-net.cs.umass.edu`)? **And the IP address of your own computer** (the host that sent the `GET`)?

<details>
<summary>💡 Show answer</summary>

`gaia.cs.umass.edu` resolves to **`128.119.245.12`** — you can read it as the **Destination IP** of your `GET` packet (or the **Source IP** of the `200 OK`). **Your computer's IP** is the **Source IP** of the `GET` — usually a private address like `192.168.x.x` or `10.x.x.x` on a home/office network.
</details>

**Q4.** Select the TCP packet that carries the HTTP `GET`. **What type of web browser** issued the request? *(Read the value after the `User-Agent:` field in the expanded HTTP section.)*

<details>
<summary>💡 Show answer</summary>

Whatever browser you used — the **`User-Agent:`** header names it. e.g. a string containing `Firefox/…`, `Safari/…`, `Chrome/…`, or `Edg/…`. This is exactly how a web server learns which browser is contacting it.
</details>

**Q5.** Expand the **Transmission Control Protocol** section of that same packet. **What is the destination port number** (after `Dest Port:`) the HTTP request was sent to?

<details>
<summary>💡 Show answer</summary>

**`80`** — the well-known port for HTTP. (Your browser's own **source** port is a random high number, e.g. `54321`.)
</details>

**Q6.** **Print the two HTTP messages** (`GET` and `200 OK`): **File → Print**, choose **"Selected Packet Only"** and **"Print as displayed"**, then **OK**.

<details>
<summary>💡 Show answer</summary>

This is a hands-on step, not a fact to look up. Select the `GET` packet, **File → Print**, tick **"Selected Packet Only"** and **"Print as displayed"**, click **OK**; then repeat for the `200 OK` packet. Tip: you can print to PDF to hand it in electronically.
</details>

---

### ➡️ Next Steps

- Apply the [Display Filter recipes](./README.md#2-essential-filter-recipes-for-network-analysis) from the README (`dns`, `icmp`, `ip.addr == …`, `tcp.port == 80`) to the *same* capture and see how the listing changes.
- Complete the [Session 1 Homework](./README.md#-homework): read the Wireshark User Guide (Ch. 1 & 3) and download a sample capture from `wiki.wireshark.org/SampleCaptures` to identify three protocols.
