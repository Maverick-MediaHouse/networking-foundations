## Lab 02 — VLAN Segmentation and Inter-VLAN Routing
**Date:** Day 2  

**Objective:**  
Verify Layer 2 VLAN isolation between two hosts, then implement 
inter-VLAN routing using Router-on-a-Stick to restore communication 
via a Layer 3 device.

**Topology:**  
PC1 (VLAN 10) — SW0 — PC2 (VLAN 20)  
SW0 trunk port — R1 (subinterfaces Gi0/0.10 and Gi0/0.20)

**IP Addressing Table:**

| Device | VLAN | IP Address | Subnet Mask | Default Gateway |
|--------|------|------------|-------------|-----------------|
| PC1 | 10 | 192.168.10.1 | 255.255.255.0 | 192.168.10.254 |
| PC2 | 20 | 192.168.20.1 | 255.255.255.0 | 192.168.20.254 |
| R1 Gi0/0.10 | 10 | 192.168.10.254 | 255.255.255.0 | — |
| R1 Gi0/0.20 | 20 | 192.168.20.254 | 255.255.255.0 | — |

**Key Commands:**
```
vlan 10
vlan 20
switchport mode access
switchport access vlan <vlan-id>
switchport mode trunk
interface gigabitEthernet 0/0.10
encapsulation dot1Q 10
ip address 192.168.10.254 255.255.255.0
interface gigabitEthernet 0/0.20
encapsulation dot1Q 20
ip address 192.168.20.254 255.255.255.0
interface gigabitEthernet 0/0
no shutdown
show vlan brief
show interfaces trunk
```

**Observations:**  
- Before adding the router, ping from PC1 to PC2 failed completely. 
  The switch enforced the VLAN boundary — frames from Fa0/1 (VLAN 10) 
  were never forwarded to Fa0/2 (VLAN 20).  
- After Router-on-a-Stick configuration, inter-VLAN routing succeeded. 
  First ping timed out due to ARP resolution for the default gateway 
  MAC. Subsequent pings succeeded.  
- TTL of received packets was 127, not 128. Windows originates ICMP 
  with TTL 128. One router hop decremented it by 1.  
- show ip route on R1 showed two connected routes and two local /32 
  entries — one set per subinterface — installed automatically when 
  Gi0/0 came up.  
- Trunk port on SW0 carried VLAN 1, 10, and 20 simultaneously using 
  802.1Q tagging.

**What This Proved:**  
VLANs are a Layer 2 broadcast domain boundary enforced by the switch. 
Communication between VLANs requires a Layer 3 device. Router-on-a-Stick 
uses 802.1Q subinterfaces to route between VLANs over a single physical 
trunk link. The trunk is mandatory — an access port would restrict the 
link to one VLAN only, breaking routing for all other VLANs.
