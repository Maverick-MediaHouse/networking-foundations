## Lab 03 — NAT and PAT
**Date:** Day 3

**Objective:**
Configure PAT on a border router to allow a private host to reach a 
public server, and verify address translation behavior using the NAT 
translation table.

**Topology:**
PC1 (private) — R1 (NAT border) — R2 (internet) — PC2 (public server)

**IP Addressing Table:**

| Device | Interface | IP Address | Subnet Mask | Role |
|--------|-----------|------------|-------------|------|
| PC1 | NIC | 192.168.1.1 | 255.255.255.0 | Private host |
| R1 | Gi0/0 | 192.168.1.254 | 255.255.255.0 | NAT inside |
| R1 | Gi0/1 | 203.0.113.1 | 255.255.255.252 | NAT outside |
| R2 | Gi0/1 | 203.0.113.2 | 255.255.255.252 | Internet router |
| R2 | Gi0/0 | 8.8.8.254 | 255.255.255.0 | Server gateway |
| PC2 | NIC | 8.8.8.1 | 255.255.255.0 | Public server |

**Key Commands:**
```
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload
interface GigabitEthernet0/0
ip nat inside
interface GigabitEthernet0/1
ip nat outside
ip route 0.0.0.0 0.0.0.0 203.0.113.2
show ip nat translations
```

**NAT Translation Table Output:**
```
Pro  Inside global     Inside local      Outside local  Outside global
icmp 203.0.113.1:5    192.168.1.1:5     8.8.8.1:5      8.8.8.1:5
icmp 203.0.113.1:6    192.168.1.1:6     8.8.8.1:6      8.8.8.1:6
icmp 203.0.113.1:7    192.168.1.1:7     8.8.8.1:7      8.8.8.1:7
icmp 203.0.113.1:8    192.168.1.1:8     8.8.8.1:8      8.8.8.1:8
```

**Field Definitions:**
- Inside local: private IP and port of the originating host
- Inside global: translated public IP and port seen by the internet
- Outside local: destination IP as seen from inside the network
- Outside global: actual destination IP on the internet

**Observations:**
- Without NAT, PC1's private IP 192.168.1.1 is unreachable from the 
  internet. R2 has no route back to 192.168.1.0/24 and would drop 
  all replies.
- PAT translated all four ICMP packets to the same public IP 
  203.0.113.1 but assigned unique port numbers per packet.
- Two initial pings timed out due to ARP resolution delay on first 
  contact.
- TTL received at PC1 was 126. PC2 originates with TTL 128. Two router 
  hops decremented it by 2 — R2 and R1 each decremented by 1.
- Default route on R1 was essential. NAT translates addresses but does 
  not make forwarding decisions. Without the default route, translated 
  packets had no path forward and were dropped at R1.
- R1 uses unique source port numbers per connection to distinguish 
  replies destined for different private hosts behind the same public IP.

**What This Proved:**
NAT and routing are independent functions — both are required for 
internet connectivity. PAT enables many private hosts to share one 
public IP by tracking connections using port numbers. The NAT table 
is the control plane record that maps every active translation. 
Private addresses are not routable on the internet — NAT is the 
mandatory boundary between private and public address space.
