# Week 3: Infrastructure & Troubleshooting

---

## Session 5 — Routing, Switching & VLANs

**Learning Objectives**: Understand how routers forward traffic. Configure VLANs. Analyze routing in captures.

### 📖 Lecture (35 min)
- **Switching**: MAC address table, broadcast vs. unicast, switch forwarding
- **VLANs**: Why segment networks? VLAN tagging (802.1Q), trunk vs. access ports
- **Routing**: Routing table, static vs. dynamic routing, default gateway
- **Inter-VLAN Routing**: Router-on-a-Stick concept, sub-interfaces
- **Subnetting Refresher**: Network/host portions, subnet mask, CIDR notation
- Live demo: `show ip route`, `show vlan brief` in Packet Tracer

### 🛠️ Hands-on Lab (45 min)

**Lab — Cisco Packet Tracer: "Small Office Network" Mega-Lab (45 min)**

> [!IMPORTANT]
> This is the core infrastructure lab of the course. Students build a realistic office network from scratch.

**Topology**: 6 PCs → 2 Switches → 1 Router → 1 Server

1. **Create 3 VLANs** on both switches:
   - VLAN 10: Admin (2 PCs)
   - VLAN 20: Sales (2 PCs)
   - VLAN 30: IT (2 PCs)
2. **Assign switch ports** to VLANs (access mode)
3. **Configure trunk port** between switches
4. **Configure Router-on-a-Stick**:
   - Sub-interfaces: `G0/0.10`, `G0/0.20`, `G0/0.30`
   - Assign gateway IPs for each VLAN
   - Enable `encapsulation dot1Q [vlan-id]`
5. **Configure DHCP** on router for each VLAN
6. **Test inter-VLAN connectivity**: Ping from Admin PC to Sales PC
7. **Verify** with: `show vlan brief`, `show ip interface brief`, `show ip route`

### 🎯 Workshop Activity: "Subnet Design Challenge" (10 min)
- Given: A company has 4 departments with 30, 60, 10, and 120 hosts
- Challenge: Design a subnetting scheme using `172.16.0.0/16`
- Each team presents their subnet plan and explains their choices
- Discuss: efficiency, scalability, and broadcast domain size

### 📚 Homework
- Save the Small Office .pkt file — it will be used in Session 6
- Research: What is OSPF and how does it differ from static routing?

---
