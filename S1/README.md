## Session 1 — Introduction to Network Analysis & The OSI Model

- [Session 1 — Introduction to Network Analysis \& The OSI Model](#session-1--introduction-to-network-analysis--the-osi-model)
  - [Lecture](#lecture)
  - [Break (10 min)](#break-10-min)
  - [Hands-on Lab](#hands-on-lab)
  - [Deep Dive: Reading Your Network Config with `ipconfig` (Before You `ping`)](#deep-dive-reading-your-network-config-with-ipconfig-before-you-ping)
    - [Reading the `ipconfig` Output Fields:](#reading-the-ipconfig-output-fields)
    - [Analyze Before You `ping`: Is the Target Local or Remote?](#analyze-before-you-ping-is-the-target-local-or-remote)
    - [The Troubleshooting Ladder: What to `ping` First](#the-troubleshooting-ladder-what-to-ping-first)
  - [Deep Dive: The `ping` Command \& Options](#deep-dive-the-ping-command--options)
    - [Analyzing the `ping` Output Fields:](#analyzing-the-ping-output-fields)
    - [Crucial `ping` Command Options:](#crucial-ping-command-options)
  - [Deep Dive: Wireshark Display Filters](#deep-dive-wireshark-display-filters)
    - [1. Common Comparison \& Logical Operators](#1-common-comparison--logical-operators)
    - [2. Essential Filter Recipes for Network Analysis](#2-essential-filter-recipes-for-network-analysis)
  - [Homework](#homework)



**Learning Objectives**: Understand what network analysis is and why it matters. Learn the OSI model and TCP/IP stack. Set up tools.

### Lecture
- **Introduction**: What is network analysis? Real-world use cases (troubleshooting, security, performance)
- **The OSI 7-Layer Model**: Detailed breakdown of the purpose and function of each layer
- **TCP/IP 4-Layer Model**: Understanding the modern mapping to OSI layers
- **Encapsulation & De-encapsulation**: Step-by-step visual walkthrough of how data becomes frames

### Break (10 min)
- Step away, stretch, and grab a drink before diving into the hands-on labs.

### Hands-on Lab

---

**Lab A — Wireshark: First Capture**

Get hands-on with **Wireshark**, the industry-standard packet sniffer. You'll install the tool, start a live capture on your network interface, load a plain-`http://` web page to generate traffic, then use a display filter to isolate the **HTTP `GET`** request and walk down its OSI layers (Ethernet → IP → TCP → HTTP) to see encapsulation for real. You'll finish by saving the capture as a `.pcap` file. A bonus **ARP activity** then shows how your computer resolves an IP to a MAC address.

> 📖 **Full step-by-step instructions, screenshots, and lab questions are in the companion guide:**
> 👉 **[Wireshark — Getting Started & Lab 1](./WIRESHARK_GUIDE.md)**

---

**Lab B — Cisco Packet Tracer: Build Your First Network**

Build a network from scratch in **Cisco Packet Tracer**, a free network simulator. You'll place two PCs and a **2901 router**, cable them up, and configure two *different* subnets — proving why a router (not just a switch) is needed to move traffic between networks. After assigning IPs and bringing up the router interfaces via the CLI, you'll verify everything with `ipconfig` and a cross-subnet `ping`, then replay it in **Simulation Mode** to watch encapsulation happen hop-by-hop. A bonus **ARP activity** reveals how a host resolves its gateway's MAC before sending across subnets.

> 📖 **Full step-by-step instructions, screenshots, and lab questions are in the companion guide:**
> 👉 **[Cisco Packet Tracer — Getting Started & Lab 2](./PACKET_TRACER_GUIDE.md)**

---

### Deep Dive: Reading Your Network Config with `ipconfig` (Before You `ping`)

Before you can sensibly `ping` anything, you need to know **your own address, your gateway, and your subnet**. The `ipconfig` family of commands prints exactly that — it's the *first* command any analyst runs to understand where a machine sits on the network.

| OS | Command | Notes |
| :--- | :--- | :--- |
| **Windows** | `ipconfig` | Add `/all` for MAC address, DNS servers, and DHCP lease info. |
| **macOS** | `ifconfig` | Or `ipconfig getifaddr en0` for just the Wi-Fi IP; `networksetup -getinfo Wi-Fi` for a summary. |
| **Linux** | `ip addr` (`ip a`) | Older systems: `ifconfig`. Use `ip route` to see the default gateway. |

#### Reading the `ipconfig` Output Fields:
```text
   IPv4 Address. . . . . . . . . . . : 192.168.1.37
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.1.1
   DNS Servers . . . . . . . . . . . : 192.168.1.1
```
* **`IPv4 Address` (`192.168.1.37`)**: *Your* host's address on the local network — the source IP of everything you send.
* **`Subnet Mask` (`255.255.255.0`)**: Splits the address into a **network part** and a **host part**. Here `/24` means the first three octets (`192.168.1`) are the network — so every `192.168.1.x` host is **local**.
* **`Default Gateway` (`192.168.1.1`)**: The **router**. Any packet whose destination is *not* on your local network is handed here to be forwarded onward.
* **`DNS Servers` (`192.168.1.1`)**: Who turns names (`google.com`) into IP addresses. If `ping <name>` fails but `ping <IP>` works, suspect DNS.

#### Analyze Before You `ping`: Is the Target Local or Remote?

Apply your subnet mask to decide *how* a packet will travel — this predicts what a `ping` actually tests:

| You are `192.168.1.37 /24`, pinging… | Same network? | Path the packet takes |
| :--- | :--- | :--- |
| `192.168.1.50` | ✅ Yes (`192.168.1.x`) | **Direct** — ARP resolves its MAC, frame sent on the local link (no router). |
| `10.0.0.5` | ❌ No | **Via the gateway** — sent to `192.168.1.1`'s MAC, then routed onward. |
| `8.8.8.8` | ❌ No | **Via the gateway → internet** — tests routing, NAT, and your ISP. |

> [!TIP]
> This is *why* the [bonus ARP activity](./WIRESHARK_GUIDE.md#activity-3--arp-who-has-this-ip) matters: for a **local** target your PC ARPs for the target itself; for a **remote** target it ARPs for the **gateway**. Reading `ipconfig` first tells you which case you're in.

#### The Troubleshooting Ladder: What to `ping` First

When connectivity breaks, `ping` outward **one rung at a time** — each step isolates a different layer, so the first rung that fails points at the problem:

1. **`ping 127.0.0.1`** (loopback) → is the **TCP/IP stack** on this machine working at all?
2. **`ping <your own IPv4>`** (e.g. `192.168.1.37`) → is your **network adapter** configured correctly?
3. **`ping <default gateway>`** (e.g. `192.168.1.1`) → can you reach the **local router**? (a Layer-1/2/local-subnet check)
4. **`ping 8.8.8.8`** (a public IP) → is **routing + NAT to the internet** working?
5. **`ping google.com`** (a name) → is **DNS** resolving names? (If 4 works but 5 fails, it's DNS, not connectivity.)

> [!NOTE]
> Climbing the ladder turns "the internet is down" into a precise diagnosis: fails at step 3 → local network issue; works to step 4 but not 5 → DNS issue.

---

### Deep Dive: The `ping` Command & Options

`ping` is the most widely used network utility for checking host reachability. It utilizes the **Internet Control Message Protocol (ICMP)**, operating at **Layer 3 (Network)** of the OSI model. 

When you run `ping`, your system sends an **ICMP Echo Request (Type 8)** packet to the target IP. If the target is alive and reachable, it responds with an **ICMP Echo Reply (Type 0)**.

#### Analyzing the `ping` Output Fields:
```text
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=14.2 ms
```
* **`64 bytes`**: The size of the ICMP payload sent.
* **`icmp_seq=1`**: The sequence number of the packet, allowing you to track packet loss and order.
* **`ttl=115` (Time to Live)**: A counter that decrements by 1 at every router hop. Used to prevent routing loops. If it hits 0, the packet is discarded.
* **`time=14.2 ms` (Round-Trip Time / RTT)**: The total milliseconds it took for the request to reach the destination and for the reply to return.

#### Crucial `ping` Command Options:

| Action / Goal | macOS / Linux Command | Windows Command | Explanation |
| :--- | :--- | :--- | :--- |
| **Set packet count** | `ping -c 5 8.8.8.8` | `ping -n 5 8.8.8.8` | Stops automatically after sending exactly `5` requests. |
| **Continuous ping** | `ping 8.8.8.8` *(default)* | `ping -t 8.8.8.8` | Pings forever until you manually stop it with **`Ctrl + C`**. |
| **Change packet size** | `ping -s 1000 8.8.8.8` | `ping -l 1000 8.8.8.8` | Sends a custom payload size (1000 bytes) to test maximum transmission unit (MTU) limits. |
| **Adjust interval** | `ping -i 0.5 8.8.8.8` | *Not native* | Sets the wait time between packets to `0.5` seconds (requires root permissions for values under `0.2`). |
| **Set Timeout** | `ping -t 2 8.8.8.8` | `ping -w 2000 8.8.8.8` | Sets a timeout in seconds (macOS) or milliseconds (Windows) before declaring a packet lost. |

---

### Deep Dive: Wireshark Display Filters

Unlike **Capture Filters** (which decide which packets are recorded to the disk *during* capture), **Display Filters** are applied to already-captured packets. They let you search, hide, and drill down into thousands of packets instantly.

Wireshark uses a very intuitive, color-coded syntax in its filter bar:
* **Green Background**: The syntax is valid.
* **Red Background**: The syntax has an error and won't run.

#### 1. Common Comparison & Logical Operators

| Operator | Alternate Syntax | Meaning | Example |
| :--- | :--- | :--- | :--- |
| **`==`** | `eq` | Equal to | `ip.addr == 192.168.1.10` |
| **`!=`** | `ne` | Not equal to | `tcp.port != 80` |
| **`&&`** | `and` | Logical AND (Both must be true) | `ip.addr == 192.168.1.10 && tcp` |
| **`\|\|`** | `or` | Logical OR (Either can be true) | `dns or http` |
| **`!`** | `not` | Logical NOT (Negate) | `!arp` (hides all ARP packets) |
| **`contains`** | *N/A* | Search for a text string | `http.host contains "example"` |

#### 2. Essential Filter Recipes for Network Analysis

| Category | Filter Expression | What it Displays |
| :--- | :--- | :--- |
| **Protocol Filters** | `http` | Shows only standard HTTP packets. |
| | `dns` | Shows all Domain Name System queries and responses. |
| | `icmp` | Shows ping requests/replies and other control messages. |
| | `arp` | Shows address resolution mapping requests/replies. |
| **IP Addresses** | `ip.addr == 192.168.1.10` | Shows any packet where this IP is either the source or destination. |
| | `ip.src == 10.0.0.5` | Shows only packets originating from this specific device. |
| | `ip.dst == 8.8.8.8` | Shows only traffic destined for Google DNS. |
| **Ports** | `tcp.port == 80` | Shows web traffic on standard HTTP port 80 (incoming or outgoing). |
| | `udp.port == 53` | Shows all standard DNS resolution port 53 traffic. |
| | `tcp.srcport == 443` | Shows outgoing encrypted HTTPS responses. |
| **Combined Filters** | `ip.addr == 192.168.1.10 && tcp.port == 80` | Isolates web traffic to/from a specific target host. |
| | `http.request.method == "GET"` | Shows only HTTP GET requests (useful for tracking web browsing). |

> [!TIP]
> **Right-Click Helper**: If you don't remember a filter name, find the field inside the **Packet Details** pane (middle pane), **right-click** it, and select **Apply as Filter** -> **Selected**. Wireshark will automatically construct the correct syntax for you!

---

### Homework
- Read: Wireshark User Guide — Chapter 1 & 3
- Explore: `wiki.wireshark.org/SampleCaptures` — download 1 PCAP and identify 3 protocols

---