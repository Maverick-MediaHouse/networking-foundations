# Network Engineering Lab Portfolio
**Author:** Sameer Mandal  
**Goal:** Junior Cloud Networking Professional — 6 Month Training Track

---

## Lab 01 — Static Routing
**Date:** Day 1  

**Objective:**  
Observe default routing table behavior, configure bidirectional static 
routing, and verify control plane route installation and removal 
triggers.

**Topology:**  
PC1 — R1 — R2 — PC2  
Two routers connected via a point-to-point /30 link.

**IP Addressing Table:**

| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| R1 | Gi0/0 (LAN) | 192.168.1.1 | 255.255.255.0 |
| R1 | Gi0/1 (WAN) | 10.0.0.1 | 255.255.255.252 |
| R2 | Gi0/1 (WAN) | 10.0.0.2 | 255.255.255.252 |
| R2 | Gi0/0 (LAN) | 192.168.2.1 | 255.255.255.0 |

**Key Commands:**
```
ip address <ip> <subnet-mask>
no shutdown
ip route 192.168.2.0 255.255.255.0 10.0.0.2
show ip route
show ip interface brief
ping <destination> source <source-ip>
```
  
**Observations:**  
- Directly connected routes are installed automatically when interfaces 
  come up — no manual configuration required.  
- Static routes are only installed when manually configured AND the 
  next-hop address is reachable.  
- When Gi0/1 was shut down, two entries were immediately removed from 
  the routing table: the directly connected 10.0.0.0/30 route and the 
  static route to 192.168.2.0/24. Both were removed because the 
  next-hop became unreachable.  
- When Gi0/1 was restored with no shutdown, all routes were 
  automatically reinstalled by the control plane.  
- First ping timed out due to ARP resolution delay. Subsequent pings 
  succeeded once the gateway MAC was cached.
- Administratively down means shutdown was issued at the control plane. 
  Down/down means the control plane is willing but Layer 1 is not 
  providing a signal.

**What This Proved:**  
Static routing requires bidirectional route configuration. Route 
removal is triggered by interface state change at the control plane — 
not by a timer or protocol. Static routes have no automatic 
reconvergence. If a link fails, traffic drops until manually 
reconfigured or a floating static route exists.

---
