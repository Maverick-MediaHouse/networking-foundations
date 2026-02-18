# Day 5 – ARP Behavior and Structured Troubleshooting
## Objective
This lab focused on understanding:
- ARP resolution behavior
- Difference between Layer 2 and Layer 3 failures
- ICMP message generation
- Interface state impact on routing
- OSPF interaction with connected routes
- Structured troubleshooting methodology

The goal was not configuration repetition, but behavioral analysis.

## Topology Overview
Multi-router OSPF topology:
PC0 — SW0 — R0 — R1 — R2 — SW2 — PC2  
With redundant routing paths already configured from previous lab.
All routers operating in OSPF Area 0.

## Phase 1 – Baseline Verification
### Commands Used

On PC0:
ipconfig
arp -a
ping 198.162.3.1

On R0:
show ip route
show ip arp

### Observations
- Baseline ping from PC0 to PC2 successful.
- ARP table on PC0 contained only default gateway MAC.
- R0 contained ARP entries for:
  - LAN host
  - Next-hop routers
- Verified routing table correctness.

## Phase 2 – ARP Cache Clearing
### Action
Cleared ARP cache on R0:
clear arp
Repeated ping from PC0.

### Observations
- No packet loss.
- Latency spikes observed (ARP resolution delay).
- ARP entries rebuilt dynamically.
- Demonstrated hop-by-hop MAC resolution.

Extended test:
Cleared ARP on:
- PC0
- R0
- R1
- R2

Observed:
- Multiple latency spikes
- No packet drops
- Confirmed sequential ARP rebuilding at each hop

## Phase 3 – Default Gateway Misconfiguration
### Action
Changed PC0 default gateway to incorrect IP (192.168.1.254).

### Result
- Ping to remote subnet failed.
- ARP request for incorrect gateway received no response.
- No packet left PC0.

### Key Learning
Setting default gateway does not assign IP ownership.
A device must have the IP configured on an interface to respond.

## Phase 4 – Interface Shutdown Analysis
### Case 1 – R0 G0/0 Shutdown
Effects:
- Connected route 192.168.1.0/24 removed from routing table.
- OSPF withdrew network advertisement.
- PC0 unable to resolve gateway MAC.
- Result: Request timed out.
- No ICMP generated (packet never reached router).

### Case 2 – R0 Has No Route to Remote Network
Effects:
- ARP to gateway succeeds.
- Packet reaches R0.
- No matching route found.
- R0 generates ICMP Destination Unreachable.
- PC0 receives explicit unreachable message.

### Case 3 – R2 LAN Interface Shutdown
Effects:
- Connected route removed from R2.
- R2 generates ICMP Destination Unreachable.
- PC0 receives unreachable message.

### Case 4 – PC2 Powered Off (Interface Up)
Effects:
- Route exists.
- R2 attempts ARP for PC2.
- No ARP reply.
- Packet dropped.
- PC0 sees Request timed out.
- No ICMP generated.

## Layer-Based Failure Comparison
| Scenario | Failure Layer | Result at PC0 |
|-----------|---------------|---------------|
| Wrong gateway | Layer 2 (ARP) | Request timed out |
| No route on router | Layer 3 | Destination unreachable |
| LAN interface down | Layer 3 (route removed) | Destination unreachable |
| Host powered off | Layer 2 (ARP at last hop) | Request timed out |

## ICMP Concepts Reinforced
- Echo Request / Echo Reply (ping)
- Destination Unreachable
- Time Exceeded (TTL)
- ICMP as IP error reporting mechanism

## Core Technical Insights
- Routing decision happens before ARP.
- ARP resolves the selected next-hop only.
- MAC address changes at every hop.
- IP address remains constant (without NAT).
- Connected routes exist only when interface state is up/up.
- OSPF withdraws networks immediately upon interface shutdown.
- ARP failure and routing failure produce different observable behaviors.

## Troubleshooting Framework Used
1. Verify IP configuration
2. Verify default gateway
3. Test local connectivity
4. Inspect ARP table
5. Inspect routing table
6. Observe ICMP feedback
7. Trace packet flow logically

## Outcome
Day 5 successfully strengthened:
- Layer 2 and Layer 3 interaction understanding
- OSPF convergence awareness
- ICMP diagnostic interpretation
- Deterministic troubleshooting approach
This lab transitioned from configuration-based learning to behavior-based network analysis.
