# Session 2 — Hands-on Lab Walkthrough

> 🌐 **English** | [日本語](./LAB_TUTORIAL.ja.md)

> **This walkthrough has moved.** To match the Session 1 layout, the labs are now split into two focused companion guides:
>
> - 🦈 **[Wireshark — Lab A: TCP & UDP in Action](./WIRESHARK_GUIDE.md)**
> - 📦 **[Packet Tracer — Lab B: Multi-Subnet IP Design Build](./PACKET_TRACER_GUIDE.md)**
>
> See the [Session 2 README](./README.md#hands-on-lab) for the lab introductions.

---

## How to add screenshots

These guides use centered, numbered figures (`Fig. N`). Each figure has a commented `<img>` line and a 📸 caption naming the file to add, e.g.:

```html
<p align="center">
  <!-- ![SYN filter applied](./img/labA-step3-synfilter.png) -->
  <em>Fig. 3 — 📸 img/labA-step3-synfilter.png: the SYN filter applied.</em>
</p>
```

To show a picture: save your screenshot into [`S2/img/`](./img/) with the **exact filename** in the caption, then **uncomment the `![...]` line** above it. It will render centered automatically.

---

## Is this enough practice for Session 2?

**Short answer: yes for the core objectives, with two suggested top-ups.**

✅ **Well covered by Labs A–B:**
- Reading real protocol headers (Ethernet / IP / TCP / UDP) — Lab A
- TCP vs. UDP behaviour (handshake vs. connectionless) — Lab A
- TCP reliability — sequence/ack numbers, segmentation, and congestion control on a graph — Lab A
- Subnet *design from scratch*, building it, and verifying routing — Lab B
- The two most common real-world failures (wrong mask, IP conflict) — Lab B

🔹 **Recommended additions for stronger retention** (each ~10–15 min, optional):
1. **Subnetting drills without a calculator.** The labs build *one* scheme; fluency comes from repetition. Treat the [homework `10.0.0.0/16` exercise](./README.md#homework) as required, not optional.
2. **A second handshake variant.** In Lab A, also capture a **TCP teardown** (`tcp.flags.fin == 1`) and an **HTTPS** session to note the payload is encrypted but the handshake is still visible.

🔸 **Deliberately deferred to later sessions:** VLAN theory & trunking → **Session 4**; DHCP/DNS/ARP services in depth → **Session 3**; routing protocols → **Session 4**.

**Verdict:** Labs A–B fully exercise every Session 2 learning objective. Add the subnetting drills (via the homework) for fluency, and the optional FIN/HTTPS capture for depth — then S2 practice is complete.

---

## Group Work — Lab B build in teams

For the **Packet Tracer Multi-Subnet IP Design Build (Lab B)**, students work in mixed teams so that every group pairs Japanese and Thai students together.

**Class composition:** 25 students total — 20 Thai (TH) + 5 Japanese (JP).

**Grouping rule:** each group has **4 TH + 1 JP = 5 students**, giving **5 groups** total. The single JP member in each group acts as the **liaison/note-taker** so the JP students are spread evenly across all teams rather than clustered.

| Group | Thai (TH) | Japanese (JP) | Size |
|---|---|---|---|
| Group 1 | TH-01, TH-02, TH-03, TH-04 | JP-01 | 5 |
| Group 2 | TH-05, TH-06, TH-07, TH-08 | JP-02 | 5 |
| Group 3 | TH-09, TH-10, TH-11, TH-12 | JP-03 | 5 |
| Group 4 | TH-13, TH-14, TH-15, TH-16 | JP-04 | 5 |
| Group 5 | TH-17, TH-18, TH-19, TH-20 | JP-05 | 5 |
| **Total** | **20** | **5** | **25** |

**How each group works:**
1. Together, design one subnet scheme for the assigned network and split the address space across the required subnets.
2. Each member builds and configures **one router/subnet** in Packet Tracer, then the group connects them into a single topology.
3. Verify end-to-end routing with `ping` / `tracert` across all subnets, and reproduce the two common failures (wrong mask, IP conflict) as a team.
4. The JP liaison records the group's final addressing table and submits it as the group deliverable.
