# Day 1 – Two Router Static Routing Lab (Cisco Packet Tracer)
## Overview
This lab demonstrates fundamental networking concepts by building a simple
two-router topology using Cisco Packet Tracer. The objective was to establish
end-to-end connectivity between two separate LANs using static routing.

This exercise focuses on **understanding**, not automation.

## Network Topology
PC0 — Switch — Router0 — Router1 — Switch — PC1

The topology contains three distinct network segments:
1. LAN 1 (PC0 ↔ Router0)
2. Point-to-point WAN link (Router0 ↔ Router1)
3. LAN 2 (Router1 ↔ PC1)

## IP Addressing Plan
### LAN 1
| Device | IP Address | Subnet |
|------|-----------|--------|
| Router0 (G0/0) | 192.168.1.1 | /24 |
| PC0 | 192.168.1.10 | /24 |

### WAN Link (Router-to-Router)
| Device | IP Address | Subnet |
|------|-----------|--------|
| Router0 (G0/1) | 10.0.0.1 | /30 |
| Router1 (G0/0) | 10.0.0.2 | /30 |

### LAN 2
| Device | IP Address | Subnet |
|------|-----------|--------|
| Router1 (G0/1) | 192.168.2.1 | /24 |
| PC1 | 192.168.2.10 | /24 |

## Why These Design Choices
### Why Subnets Exist
Subnets create logical boundaries that define broadcast domains and control
traffic flow. Devices within a subnet can communicate directly, while traffic
between subnets must pass through a router.

### Why /30 Was Used Between Routers
The router-to-router link only requires two usable IP addresses.
A /30 subnet provides:
- 4 total addresses
- 2 usable host addresses
This avoids IP wastage and reflects real-world network design best practices.

### Why Static Routes Were Required
Routers only know:
- Directly connected networks
- Networks explicitly added to their routing table
Since each router was unaware of the other LAN, static routes were added to
define explicit paths.

### Why Default Gateways Matter
End devices (PCs) cannot route traffic.
If a destination is outside the local subnet, traffic must be sent to a
default gateway. An incorrect or missing gateway prevents packets from ever
leaving the LAN.

## Static Routing Configuration
### Router0
ip route 192.168.2.0 255.255.255.0 10.0.0.2

### Router1
ip route 192.168.1.0 255.255.255.0 10.0.0.1

## Verification Steps
1. Verified interface status using: show ip interface brief
2. Verified routing tables using: show ip route
3. Performed hop-by-hop ping tests
4. Confirmed end-to-end connectivity between PC0 and PC1

## Debugging Insight

Initial ping failures were caused by incorrect IP configuration on PC1.
Routing was correct, but traffic failed due to host-side misconfiguration.
##### Key lesson:
> If routing tables are correct, always verify host configuration first.

## Key Takeaways
- Subnet boundaries define routing behavior
- Routers require explicit routing information
- Proper subnet sizing is critical
- Host configuration errors can silently break networks

## Tools Used
- Cisco Packet Tracer
- Static Routing (IPv4)

## Next Steps
- Replace static routing with a dynamic protocol (OSPF)
- Introduce additional LAN segments
- Apply the same logic to cloud networking concepts (VPCs, route tables)
