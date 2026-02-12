# Day 2 – Dynamic Routing with OSPF (Cisco Packet Tracer)
## Overview
This lab replaces static routing with OSPF (Open Shortest Path First) to demonstrate
dynamic route discovery and automatic path calculation between two routers.

The objective was to understand:
- Why static routing does not scale
- How dynamic routing protocols exchange network information
- How OSPF builds routing tables automatically

## Topology
PC0 — Switch — Router0 — Router1 — Switch — PC1

Three network segments:
1. LAN 1 → 192.168.1.0/24  
2. WAN Link → 10.0.0.0/30  
3. LAN 2 → 192.168.2.0/24  

## Why Static Routing Was Removed

Static routes require manual configuration on each router.
Limitations:
- Does not scale
- Requires manual updates for topology changes
- Error-prone in larger networks
- No automatic failure handling

Dynamic routing solves these problems.

## OSPF Configuration
### Router0
router ospf 1
network 192.168.1.0 0.0.0.255 area 0
network 10.0.0.0 0.0.0.3 area 0

### Router1
router ospf 1
network 192.168.2.0 0.0.0.255 area 0
network 10.0.0.0 0.0.0.3 area 0

## What These Commands Do
- `router ospf 1`  
  Starts OSPF process ID 1 (locally significant)
- `network X wildcard area 0`  
  Activates OSPF on matching interfaces  
  Advertises connected networks  
  Places interfaces in OSPF Area 0 (backbone area)
Wildcard masks are inverse subnet masks:
- /24 → 0.0.0.255
- /30 → 0.0.0.3

## Verification
### Neighbor Formation
show ip ospf neighbor

Expected state: `FULL`

### Routing Table
show ip route

Static routes (`S`) were replaced with:
O 192.168.x.x
`O` indicates routes learned dynamically via OSPF.

## Key Differences: Static vs Dynamic Routing
| Static Routing | OSPF |
|---------------|------|
| Manual path configuration | Automatic route exchange |
| No failure adaptation | Automatic recalculation |
| Poor scalability | Designed for large networks |
| High human dependency | Protocol-driven intelligence |

## Debugging Insight
After removing static routes, connectivity failed as expected.
Once OSPF adjacency formed, routing tables populated automatically,
and end-to-end communication was restored.

Key lesson:
Dynamic routing requires successful neighbor adjacency.
If adjacency fails, no routes are exchanged.

## Why Area 0 Is Used
OSPF requires a backbone area (Area 0).
All other OSPF areas must connect to Area 0.
In small labs, a single-area OSPF design is sufficient.

## Key Takeaways
- OSPF automates route distribution.
- Routers build topology awareness using LSAs.
- Routing tables are calculated using cost metrics.
- Dynamic routing enables scalability and fault tolerance.

## Tools Used
- Cisco Packet Tracer
- OSPF (Open Shortest Path First)
- IPv4 Routing

## Next Steps
- Introduce a third router
- Observe OSPF recalculation on link failure
- Compare OSPF with EIGRP
- Apply concepts to cloud route tables (AWS / Azure)
