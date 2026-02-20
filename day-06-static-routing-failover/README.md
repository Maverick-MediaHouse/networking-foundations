# Day 6 – Static Routing, Floating Routes, Blackhole Behavior & Control Plane Analysis
## Objective
To understand:
- Static routing behavior
- Administrative Distance (AD)
- Floating static routes
- Recursive next-hop resolution
- Blackhole routing
- Interface-state vs reachability-based failover

## Topology Overview
- R0 ↔ R1 ↔ R2 (Primary Path)
- R0 ↔ R3 ↔ R2 (Backup Path)
- LAN 192.168.1.0/24 behind R0
- LAN 192.168.3.0/24 behind R2

## Step 1 – Disable OSPF
no router ospf 1

Verification:
show ip route 192.168.1.0

Only connected routes should remain.

## Step 2 – Configure Static Routes
### On R0
Primary route:
ip route 192.168.3.0 255.255.255.0 10.0.0.2

Floating backup route:
ip route 192.168.3.0 255.255.255.0 10.0.0.10 200

### On R2 (Return Path)
Primary:
ip route 192.168.1.0 255.255.255.0 10.0.0.5

Backup:
ip route 192.168.1.0 255.255.255.0 10.0.0.13 200

## Step 3 – Verify Routing Table

Command:
show ip route 192.168.1.0

Observed behavior:
- Only lowest AD (1) route installed
- Backup (AD 200) remains in configuration but not in routing table
- Recursive lookup successful

## Step 4 – Interface-Based Failover Test

Action:
- Shutdown primary interface

Observed:
- Connected route removed
- Recursive resolution failed
- Primary static removed
- Floating static installed immediately
- Minimal packet loss

Conclusion:
Failover triggered by interface state change is immediate and deterministic.

## Step 5 – Blackhole Routing Test
Scenario:
- Remote router powered off
- Local interface remains UP

Observed:
- Static route remains installed
- Traffic forwarded
- No replies received
- Ping fails

Conclusion:
Static routing does not validate remote reachability.
This creates blackhole routing conditions.

## Key Concepts Learned

| Concept | Explanation |
|----------|-------------|
| Static Route Removal | Depends on interface state or recursive resolution |
| Floating Static | Higher AD used as backup |
| Recursive Lookup | Next-hop must be reachable |
| Blackhole Routing | Route installed but traffic silently dropped |
| Control vs Data Plane | Routing table logic separate from ARP behavior |

## Engineering Insight

Static routing is deterministic but blind to remote device health.
In production environments, engineers use:
- IP SLA
- Object Tracking
- Dynamic routing protocols

To achieve reachability-aware failover.

## Summary

Day 6 focused on understanding routing behavior at control-plane depth rather than simply configuring commands.

The lab demonstrated:
- Administrative Distance selection
- Interface-triggered failover
- Limitations of static routing
- Real-world blackhole scenarios

This builds foundational understanding for dynamic routing protocols and enterprise network design.
