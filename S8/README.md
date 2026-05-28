## Session 8 — Capstone Project & Course Review

**Learning Objectives**: Apply all skills in a comprehensive scenario. Demonstrate proficiency. Explore career paths.

### 📖 Lecture & Review (25 min)
- Quick review: OSI model → Protocols → Services → Routing → Troubleshooting → Security
- **Beyond Wireshark**:
  - `tcpdump` — capture on headless servers: `tcpdump -i eth0 -w capture.pcap`
  - `tshark` — CLI Wireshark: `tshark -r file.pcap -Y "http.request"`
- **Career Paths & Certifications**:
  - CompTIA Network+ (N10-009)
  - CompTIA Security+ (SY0-701)
  - GIAC Certified Intrusion Analyst (GCIA)
  - Cisco CCNA
- **Continuing Education Resources**:
  - TryHackMe, Blue Team Labs Online, CyberDefenders
  - Malware-Traffic-Analysis.net (advanced PCAPs)
  - Chris Greer's YouTube (Wireshark tutorials)

### 🛠️ Capstone Challenge (50 min)

> [!IMPORTANT]
> The capstone is a practical, scenario-based assessment that covers the entire course.

**Scenario**: *"A company reports that employees cannot access the internal website, and suspicious traffic has been detected on the network."*

**Part 1 — Packet Tracer: Fix the Network (20 min)**
- Students receive a broken network topology
- Must restore connectivity by fixing routing, VLAN, and DHCP issues
- Must implement one ACL as a security measure

**Part 2 — Wireshark: Incident Analysis (20 min)**
- Students receive a PCAP from the "compromised" network
- Must answer:
  1. What services are running on the network?
  2. Identify the attacker's IP address
  3. What type of scan was performed?
  4. Were any credentials compromised?
  5. Was any data exfiltrated? If so, recover it.
- Write a 1-page **Incident Report** with findings and recommendations

**Part 3 — Presentation (10 min)**
- Each student/team presents their top finding to the class
- Peer discussion and feedback

### 🎯 Workshop Activity: "Build & Break" Competition (remaining time)
- **Phase 1**: Each team builds a small network in Packet Tracer (5 min)
- **Phase 2**: Teams swap — each team must find and fix the other team's hidden problems
- Awards for: Best Network Design, Fastest Fixer, Most Creative Problem