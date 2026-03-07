IPv4 Header format:
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |     DSCP      |          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|    Fragment Offset      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      TTL      |   Protocol    |        Header Checksum        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Source IP                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      Destination IP                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Version — 4 bits. Value is 4 for IPv4, 6 for IPv6. Router reads this first to know how to process the packet.
IHL — Internet Header Length — 4 bits. Tells the router where the header ends and data begins. Minimum value is 5, meaning 20 bytes. This matters because options can extend the header.
DSCP — Differentiated Services Code Point — 8 bits. Quality of Service marking. Tells routers how to prioritize this packet. Voice traffic gets higher priority than email. This is control plane input for traffic engineering.
Total Length — 16 bits. Size of the entire packet including header and data. Maximum 65,535 bytes.
Identification — 16 bits. Unique ID assigned to each original packet. If fragmentation occurs, all fragments of the same original packet share the same identification number. This is how the receiving end knows which fragments belong together.
Flags — 3 bits. Controls fragmentation behavior. The important ones: DF bit means Do Not Fragment — if set and the packet is too large for the next link, the router drops it and sends an ICMP message back. MF bit means More Fragments — tells the receiver more fragments are coming.
Fragment Offset — 13 bits. Tells the receiver where this fragment belongs in the original packet. Measured in 8-byte units.
TTL — Time To Live — 8 bits. You already know this. Every router decrements by 1. Reaches zero, packet dropped, ICMP Time Exceeded sent back to source. Prevents infinite loops.
Protocol — 8 bits. Tells the receiving device what is inside the payload. Value 1 means ICMP. Value 6 means TCP. Value 17 means UDP. This is how the data plane knows which upper layer process to hand the packet to.
Header Checksum — 16 bits. Error detection for the header only. Every router recalculates it after decrementing TTL. If checksum fails, packet is silently dropped.
Source IP — 32 bits. Where the packet came from. NAT modifies this field on outbound packets.
Destination IP — 32 bits. Where the packet is going. This is what the router uses for longest prefix match lookup.

The control plane vs data plane split in the header:
Control plane uses: Destination IP for routing decisions, TTL for loop prevention, Protocol for upper layer handling, Flags for fragmentation decisions.
Data plane uses: everything else to actually process and forward the packet.

Practical IPv4 Header Walkthrough — PC1 to PC2 through two routers:
Topology:
PC1 (192.168.1.1) — R1 (203.0.113.1) — R2 (203.0.113.2) — PC2 (8.8.8.1)

Step 1 — PC1 creates the packet:
Source IP:      192.168.1.1
Destination IP: 8.8.8.1
TTL:            128  (Windows default)
Protocol:       1    (ICMP)
Checksum:       calculated over all header fields
PC1 checks: is 8.8.8.1 in my subnet 192.168.1.0/24? No. So it sends the frame to its default gateway R1 at 192.168.1.254. Destination MAC in the frame is R1's MAC. Destination IP in the packet is still 8.8.8.1 — unchanged.

Step 2 — R1 receives the packet:
Control plane actions in order:
One — strip the Layer 2 frame. Look at the IP header.
Two — run longest prefix match on destination 8.8.8.1. Matches default route 0.0.0.0/0 via 203.0.113.2.
Three — NAT is configured. R1 checks access-list 1. 192.168.1.1 matches. Translate source IP.
Four — modify header:
Source IP:      192.168.1.1 → 203.0.113.1  (NAT translation)
Destination IP: 8.8.8.1                     (unchanged)
TTL:            128 → 127                   (decremented by 1)
Checksum:       recalculated                (source IP and TTL both changed)
Protocol:       1                           (unchanged)
Five — build new Layer 2 frame with R2's MAC as destination. Forward out Gi0/1.

Step 3 — R2 receives the packet:
Control plane actions in order:
One — strip Layer 2 frame. Look at IP header.
Two — longest prefix match on 8.8.8.1. Matches connected route 8.8.8.0/24 via Gi0/0.
Three — no NAT on R2. No translation needed.
Four — modify header:
Source IP:      203.0.113.1   (unchanged)
Destination IP: 8.8.8.1       (unchanged)
TTL:            127 → 126     (decremented by 1)
Checksum:       recalculated  (TTL changed)
Protocol:       1             (unchanged)
Five — ARP for 8.8.8.1's MAC. Build new frame. Forward out Gi0/0.

Step 4 — PC2 receives the packet:
PC2 sees:
Source IP:      203.0.113.1  (R1's public IP — PC2 never sees 192.168.1.1)
Destination IP: 8.8.8.1
TTL:            126
Protocol:       1 — process as ICMP
PC2 generates ICMP echo reply. Source becomes 8.8.8.1, destination becomes 203.0.113.1. Sends it back.

Step 5 — Return path hits R1:
R1 receives reply destined for 203.0.113.1. Checks NAT table. Finds translation entry. Rewrites destination IP from 203.0.113.1 back to 192.168.1.1. Recalculates checksum. Forwards to PC1.
PC1 receives the reply and sees it came from 8.8.8.1. It has no idea NAT happened.

The pattern at every single hop is always:
One — TTL decremented. Two — checksum recalculated. Three — Layer 2 frame rebuilt with new MACs. Four — IP header destination unchanged except when NAT modifies it.
The IP header destination IP never changes hop to hop unless NAT is involved. The Layer 2 MAC addresses change at every single hop. That distinction is fundamental.
