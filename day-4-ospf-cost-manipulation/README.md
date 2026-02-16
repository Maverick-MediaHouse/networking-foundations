# Day 4 – OSPF Cost Manipulation and ECMP
## Objective
Understand how OSPF selects paths using cost and how bandwidth manipulation influences routing decisions.

## Topology
Redundant square topology:
Path 1:
R0 → R1 → R2

Path 2:
R0 → R3 → R2
All routers configured in OSPF Area 0.

## IP Plan

R0–R1 → 10.0.0.0/30  
R1–R2 → 10.0.0.4/30  
R0–R3 → 10.0.0.8/30  
R3–R2 → 10.0.0.12/30  

R2 LAN → 192.168.3.0/24

## Step 1 – ECMP Observation

Before bandwidth manipulation:
show ip route 192.168.3.0

Two next hops appeared.

Reason:
Both paths had equal cumulative OSPF cost.

This demonstrates Equal Cost Multi Path (ECMP).

## Step 2 – Bandwidth Manipulation

On R0:

interface GigabitEthernet0/2
bandwidth 10

This increased OSPF cost on R0–R3 link.
New cost calculation:
Cost = Reference Bandwidth / Interface Bandwidth

Result:
Path via R3 became significantly more expensive.

## Step 3 – Route Recalculation

After manipulation:
show ip route 192.168.3.0

Only one next hop remained.
OSPF recalculated SPF and selected the lowest-cost path.

## Key Concepts Learned

- OSPF cost formula
- Reference bandwidth impact
- Cumulative metric calculation
- Equal Cost Multi Path (ECMP)
- Administrative Distance vs Metric

## Technical Insight
OSPF does not count hops.
It calculates cumulative interface costs using SPF.

Bandwidth directly influences routing decisions.
