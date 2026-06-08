# 🛠️ Session 2 — Hands-on Lab Walkthrough

> **This walkthrough has moved.** To match the Session 1 layout, the labs are now split into two focused companion guides:
>
> - 🦈 **[Wireshark — Lab A: The TCP 3-Way Handshake](./WIRESHARK_GUIDE.md)**
> - 📦 **[Packet Tracer — Labs B & C: Protocol Walk + Multi-Subnet Design](./PACKET_TRACER_GUIDE.md)**
>
> See the [Session 2 README](./README.md#️-hands-on-lab) for the lab introductions.

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

✅ **Well covered by Labs A–C:**
- Reading real protocol headers (Ethernet / IP / TCP / UDP) — Lab A + Lab B
- TCP vs. UDP behaviour (handshake vs. connectionless) — Lab A
- Encapsulation and ARP hop-by-hop — Lab B
- Subnet *design from scratch*, building it, and verifying routing — Lab C
- The two most common real-world failures (wrong mask, IP conflict) — Lab C

🔹 **Recommended additions for stronger retention** (each ~10–15 min, optional):
1. **Subnetting drills without a calculator.** The labs build *one* scheme; fluency comes from repetition. Treat the [homework `10.0.0.0/16` exercise](./README.md#-homework) as required, not optional.
2. **A second handshake variant.** In Lab A, also capture a **TCP teardown** (`tcp.flags.fin == 1`) and an **HTTPS** session to note the payload is encrypted but the handshake is still visible.

🔸 **Deliberately deferred to later sessions:** VLAN theory & trunking → **Session 4**; DHCP/DNS/ARP services in depth → **Session 3**; routing protocols → **Session 4**.

**Verdict:** Labs A–C fully exercise every Session 2 learning objective. Add the subnetting drills (via the homework) for fluency, and the optional FIN/HTTPS capture for depth — then S2 practice is complete.
