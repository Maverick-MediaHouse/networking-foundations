# Day 3 – Multi-Router OSPF Topology & Reconvergence Analysis
## Objective
Expand a 2-router OSPF topology into a 3-router network, implement dynamic routing using OSPF, and observe protocol behavior during link failure and reconvergence.

## Topology
```
PC0 — R0 — R1 — R2 — PC2
            |
           PC1
```
### Devices Used

* 3 Routers (R0, R1, R2)
* 3 Switches
* 3 PCs

## IP Addressing Scheme
### Router-to-Router Links (/30)

| Link    | Subnet      | IP Assignment   |
| ------- | ----------- | --------------- |
| R0 – R1 | 10.0.0.0/30 | R0: .1 / R1: .2 |
| R1 – R2 | 10.0.0.4/30 | R1: .5 / R2: .6 |

### LAN Networks (/24)

| Router | Subnet         | Gateway IP  | Host Example |
| ------ | -------------- | ----------- | ------------ |
| R0     | 192.168.1.0/24 | 192.168.1.1 | 192.168.1.2  |
| R1     | 192.168.2.0/24 | 192.168.2.1 | 192.168.2.2  |
| R2     | 192.168.3.0/24 | 192.168.3.1 | 192.168.3.2  |

## OSPF Configuration (Area 0)
Example configuration for R2:
```
router ospf 1
 network 10.0.0.4 0.0.0.3 area 0
 network 192.168.3.0 0.0.0.255 area 0
```
All routers were configured in Area 0 (Backbone Area).

## Key Observations
### 1. OSPF Neighbor Formation
Command:
```
show ip ospf neighbor
```
* R1 formed adjacency with both R0 and R2.
* State reached FULL after LSDB synchronization.

### 2. Route Propagation

After enabling OSPF:
* R0 learned 192.168.3.0/24 via R1.
* Route code displayed as `O` (OSPF).
* No static routes were required.

Command:

```
show ip route
```
### 3. Control Plane vs Data Plane
* OSPF LSDB stores full topology.
* Routing table stores best paths calculated using SPF.
* Administrative Distance determines protocol preference.

### 4. Failure Simulation & Reconvergence
When R1–R2 link was shut down:
```
interface g0/x
 shutdown
```
Observed behavior:

* OSPF adjacency dropped.
* 10.0.0.4/30 removed from routing table.
* 192.168.3.0 route disappeared from R0 and R1.
* End-to-end ping failed.

After `no shutdown`:

* Adjacency re-established.
* LSDB synchronized.
* Routes reinstalled.
* Connectivity restored.
This demonstrates dynamic reconvergence.

## Concepts Reinforced

* CIDR and /30 subnet design
* OSPF neighbor states
* LSDB vs Routing Table
* SPF (Dijkstra) path calculation
* Administrative Distance vs Metric
* Link failure detection via Hello/Dead timers
* Automatic topology recalculation

## Key Engineering Insight
Static routing scales poorly because it requires manual updates during topology changes.
OSPF dynamically maintains a synchronized topology database and recalculates optimal paths automatically, enabling scalable and resilient network design.

## Skills Practiced
* Subnet planning
* Multi-router configuration
* OSPF deployment
* Troubleshooting adjacency failures
* Diagnosing route propagation issues
* Failure simulation and recovery testing

## Outcome
Successfully implemented a 3-router OSPF network with dynamic route exchange and validated protocol reconvergence during link failure.
This lab demonstrates foundational understanding of link-state routing and scalable network design principles.
