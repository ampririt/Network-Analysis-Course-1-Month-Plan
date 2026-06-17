# 🌎Network Analysis Course — 1-Month Plan

> 🌐 **English** | [日本語](./README.ja.md)

> **Duration**: 4 Weeks | **Schedule**: 1 Class/Week | **Class Length**: 3 Hours (including 10 min break)  
> **Total Sessions**: 4 | **Total Contact Hours**: 12 Hours

---

## Course Overview

This course introduces students to the fundamentals of network analysis — from understanding how data flows across networks to capturing, inspecting, and troubleshooting real traffic. Students will gain hands-on experience with industry-standard tools (Wireshark, Cisco Packet Tracer) and develop practical skills applicable to IT support, network administration, and cybersecurity careers.

## Prerequisites

- Basic computer literacy (file management, installing software)
- Conceptual understanding of the internet
- Command-line familiarity is helpful but **not required**

## Required Software (Free)

| Tool | Purpose | Download Link & Installation Guide |
|------|---------|----------|
| **Wireshark** | Live packet capture & protocol analysis | [wireshark](https://www.wireshark.org/docs/wsug_html_chunked/ChBuildInstallWinInstall.html) |
| **Cisco Packet Tracer** | Network simulation & topology design | [netacad](https://www.netacad.com/skillsforall/files/Cisco_Packet_Tracer_Download_and_Installation_Instructions.pdf) |

## Table of Topic
- [Session 1 — Introduction to Network Analysis & The OSI Model](/S1/README.md)
- [Session 2 — The Protocol Stack Deep Dive: Ethernet, IP, TCP & UDP](/S2/README.md)
- [Session 3 — Network Services: DHCP, DNS & ARP](/S3/README.md)
- [Session 4 — Routing, Switching & VLANs](/S4/README.md)

---

## Bonus — Interactive Resources

Standalone, browser-based extras to explore on your own — just open the HTML file, no install needed:

| Resource | What it is |
|----------|-----------|
| **Network Fundamentals Simulator**<br>(network_simulation.html) | An interactive playground for the core concepts — **OSI encapsulation**, the **TCP 3-way handshake**, a packet's journey, **DNS/ARP** walkthroughs, and **IP/IPv6/subnet** calculators. *(Japanese UI)* |
| **PacketLens — Packet Analyzer**<br>(wireshark_explain/index.html) | A Wireshark-style "signal analyzer" that teaches **how to read packets**: capture → analyze, **protocol dissection** (HTTP / TCP / DNS), packet stream, and **hex · ASCII** field views — all from safe simulated captures (no bytes on the wire). |
| **Shell & Scripting for Network Engineers**<br>(shell_scripting_for_network_engineers.html) | A 53-slide interactive deck on automating the command line in **Bash & PowerShell** side by side — network commands (`ping`, `dig`, `netstat`, `curl`), pipes & filters (`grep`, `awk`, `tee`), writing your first script step by step, loops / conditionals / functions, a 5-script toolkit, and **cron / Task Scheduler**. Click any code to explain it; adjustable text size. |

---

## Time Allocation Per Session (3 Hours / 180 Minutes)

| Segment | Duration | Description |
|---------|----------|-------------|
| 🔄 Warm-up & Review | 10 min | Recap previous session, Q&A, and homework check |
| 📖 Lecture & Live Demo | 50 min | Core concepts with real-world, real-time demonstrations |
| ☕ Break | 10 min | Stretch, stand up, and grab a drink before the lab |
| 🛠️ Hands-on Lab | 80 min | In-depth guided exercises using Wireshark & Cisco Packet Tracer |
| 📝 Wrap-up & Preview | 10 min | Key takeaways, homework overview, and next session preview |


---

<!-- ## Assessment Summary

| Assessment | Weight | Session |
|------------|--------|---------|
| Class Participation & Labs | 30% | Every session |
| Homework Assignments | 20% | Weekly |
| Mini CTF Performance | 15% | Session 3 |
| Capstone Project | 35% | Session 4 |

--- -->

## Recommended Resources
> [!IMPORTANT] Importance
> You can check more information about Wireshark at this [Wireshark User’s Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
### PCAP Datasets for Practice
| Source | URL | Level |
|--------|-----|-------|
| Wireshark Sample Captures | `wiki.wireshark.org/SampleCaptures` | Beginner |
| Netresec PCAP Repository | `netresec.com` | Intermediate |
| Malware-Traffic-Analysis.net | `malware-traffic-analysis.net` | Advanced |
| Digital Corpora | `digitalcorpora.org` | Intermediate |

### Lab Resources
| Source | URL | Description |
|--------|-----|-------------|
| Kurose & Ross Wireshark Labs | UMass textbook site | Gold-standard protocol labs |
| Jeremy's IT Lab | YouTube + GitHub | 76+ Packet Tracer labs |
| Cisco NetAcad | `netacad.com` | Official structured courses |
| TryHackMe | `tryhackme.com` | Interactive security rooms |
| Blue Team Labs Online | `blueteamlabs.online` | Defensive challenges |

---


> [!NOTE]
> **Lab Environment**: students should have their own laptops with admin rights to install Wireshark and Packet Tracer.




