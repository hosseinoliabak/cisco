# Protocol Independent Multicast (PIM)

Subscribers use IGMP to join a multicast group. If the publisher also resides in the same network as the subscriber, it is sufficient. However, if the subscriber and the publisher are not in the same network, a multicast routing protocol must exist to route the multicast traffic down to the subscriber from the source.

Cisco NX-OS supports PIM sparse-mode (SM) as the multicast routing protocol. NX-OS does not support PIM Dense Mode. So, We won’t cover PIM-DM throughout the CCIE Datacenter course.

PIM-SM builds:
  * RPT: multiple source
  * SPT: single source

All PIM control messages use IP Protocol number 103. The control messages are either unicast or multicast. If the control message is multicast the address is 224.0.0.13 (All PIM Routers).

### PIM Sparse-Mode

Subscribers send an IGMP join to the LHR. This IGMP join causes the LHR to send a PIM join (*,G) to the root of the tree. (With RPT, the root is RP. With SPT, the root is FHR). The RP sends a PIM join (S,G) to the FHR. With that being said, two trees are created: an SPT from FHR to RP and an RPT from RP to LHR. At this point, the traffic flows from source to the RP and from the RP to LHR.

#### Multicast Forwarding in PIM SM

FHR sends the multicast traffic to RP. RP at this point will have the (S,G) entry. FHR sends this multicast traffic in a special unicast message called PIM Register using a unidirectional PIM tunnel.

* When RP receives this message, it de-encapsulates the multicast traffic inside the register message, and
  * if there is no subscriber, the RP sends a PIM Register Stop back to the FHR without traversing the PIM runnel telling it to stop sending register messages.
  * if there is subscriber for that group, RP forwards the multicast traffic down the (*,G) RPT and it also sends a join back towards the FHR to create an (S,G) SPT.
    * Once SPT forms, the RP sends a register stop message to FHR so that it could stop unicast register messages. At this point multicast traffic is forwarded throughout the network. However, that unidirectional PIM register tunnel stays up/up and remain active as long as RPF check succeeds.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235807644-de05bf25-2aa2-4ead-9e46-da527952450d.png" alt="PIM-SM Multicast Distribution Tree Building">
  <figcaption>Figure 1: PIM-SM Multicast Distribution Tree Building</figcaption>
</figure>
<p>&nbsp</p>   

* Once the LHR receives the first multicast traffic from RP, it knows the source IP address of the multicast traffic. LHR checks the unicast routing table to find the best path to the source. It then sends a (S,G) PIM join to the FHR, and sends a PIM prune message to the RP. This is called PIM SPT Switchover which is default behavior in Cisco NX-OS.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235807792-0c56abcd-4a8a-441f-92e3-882ceec903d5.png" alt="PIM-SM SPT Switchover">
  <figcaption>Figure 2: PIM-SM SPT Switchover</figcaption>
</figure>
<p>&nbsp</p>   


#### Designated Router

Note that when multiple routers exists on the publisher and subscriber LAN segments, Designated Router is:

  * on FHR responsible for sending PIM register and forwarding the multicast traffic.
  * on LHR responsible for sending PIM join and prune messages.

If the DR priority of the PIM routers are equal (by default = 1), the highest IP address in the segment is DR.

There is a combination of three modes which NX-OS supports for different range of multicast groups.
  * Any-source multicast (ASM)
  * Bidirectional shared trees (Bidir)
  * Source-specific multicast (SSM)

### Any-source multicast (ASM)

PIM-SM is commonly referred to as any-source multicast (ASM). ASM mode requires an RP for a group. When you configure the RP, the ASM is default mode on NX-OS.

There are two trees:
  * Shared tree roots at RP
  * Source tree roots at FHR

```c
N9K01# configure
Enter configuration commands, one per line. End with CNTL/Z.
N9K01(config)# feature pim
N9K01(config)# int e1/1
N9K01(config-if)# ip pim sparse-mode
N9K01(config)# ip pim rp-address 192.168.0.254
```
```c
N9K03# show ip pim rp
PIM RP Status Information for VRF "default"
BSR disabled
Auto-RP disabled
BSR RP Candidate policy: None
BSR RP policy: None
Auto-RP Announce policy: None
Auto-RP Discovery policy: None

RP: 192.168.0.254, (0),
 uptime: 00:08:17   priority: 255,
 RP-source: (local),
 group ranges:
 224.0.0.0/4

N9K01# show ip mroute
IP Multicast Routing Table for VRF "default"

(*, 232.0.0.0/8), uptime: 00:34:05, pim ip
  Incoming interface: Null, RPF nbr: 0.0.0.0
  Outgoing interface list: (count: 0)


(*, 239.0.1.2/32), uptime: 00:01:34, pim ip
  Incoming interface: loopback1, RPF nbr: 192.168.0.254
  Outgoing interface list: (count: 1)
    Ethernet1/2, uptime: 00:01:34, pim


(172.16.1.2/32, 239.0.1.2/32), uptime: 00:08:02, pim ip
  Incoming interface: Ethernet1/1, RPF nbr: 10.1.1.1, internal
  Outgoing interface list: (count: 1)
    Ethernet1/2, uptime: 00:01:34, pim


(*, 239.255.255.250/32), uptime: 00:01:41, pim ip
  Incoming interface: loopback1, RPF nbr: 192.168.0.254
  Outgoing interface list: (count: 0)


N9K04# show ip igmp groups
IGMP Connected Group Membership for VRF "default" - 1 total entries
Type: S - Static, D - Dynamic, L - Local, T - SSM Translated, H - Host Proxy
      * - Cache Only
Group Address      Type Interface              Uptime    Expires   Last Reporter
239.0.1.2          D   Ethernet1/7            00:00:28  00:03:51  172.16.4.2

N9K04(config-if)# show ip mroute
IP Multicast Routing Table for VRF "default"

(*, 232.0.0.0/8), uptime: 00:35:39, pim ip
  Incoming interface: Null, RPF nbr: 0.0.0.0
  Outgoing interface list: (count: 0)


(*, 239.0.1.2/32), uptime: 00:11:42, igmp ip pim
  Incoming interface: Ethernet1/1, RPF nbr: 10.1.1.2
  Outgoing interface list: (count: 1)
    Ethernet1/7, uptime: 00:11:42, igmp
```

To simulate multicast source and destination, I have used two Cisco IOS-XE routers. Here is their configuration:

```c
SUBSCRIBER-1(config)#ip multicast-routing
SUBSCRIBER-1(config)#interface Ethernet0/0
SUBSCRIBER-1(config-if)#ip address 172.16.4.2 255.255.255.0
SUBSCRIBER-1(config-if)#ip pim dr-priority 0
SUBSCRIBER-1(config-if)#ip pim sparse-mode
SUBSCRIBER-1(config-if)#ip igmp join-group 239.0.1.2
!
PUBLISHER-1(config)#ip multicast-routing
PUBLISHER-1(config)#interface Ethernet0/0
PUBLISHER-1(config-if)#ip address 172.16.1.2 255.255.255.0
PUBLISHER-1(config-if)#ip pim dr-priority 0
PUBLISHER-1(config-if)#ip pim sparse-mode
PUBLISHER-1(config-if)#end

PUBLISHER-1#ping 239.0.1.2 repeat 4
Type escape sequence to abort.
Sending 4, 100-byte ICMP Echos to 239.0.1.2, timeout is 2 seconds:
Reply to request 0 from 172.16.4.2, 15 ms
Reply to request 1 from 172.16.4.2, 11 ms
Reply to request 2 from 172.16.4.2, 10 ms
Reply to request 3 from 172.16.4.2, 10 ms
```

You can define the RPs statically or dynamically using Auto-RP (Cisco proprietary) or BSR (Standard RFC 5059). If an RP is defined, the group operates in ASM mode.

### Bidirectional shared trees (Bidir)

BiDir PIM is like ASM with that you still need RP. But you only have (*,G) state. So, you won’t have PIM register and (S,G) entries. This means that the multicast traffic always flows to the RP. Designated Forwarder is loop prevention mechanism. For the configuration you add bidir keyword to make the RP in bidir mode:

```c
! On everyone issue ip pim rp-address 192.168.0.254 bidir
N9K03(config)# ip pim rp-address 192.168.0.254 bidir
N9K03# show ip mroute
IP Multicast Routing Table for VRF "default"

(*, 224.0.0.0/4), bidir, uptime: 00:01:05, pim ip
  Incoming interface: Ethernet1/1, RPF nbr: 10.1.1.0
  Outgoing interface list: (count: 1)
    Ethernet1/1, uptime: 00:01:05, pim, (RPF)

(*, 232.0.0.0/8), uptime: 00:01:52, pim ip
  Incoming interface: Null, RPF nbr: 0.0.0.0
  Outgoing interface list: (count: 0)

! No (S,G) anymore
```

### Source-specific multicast (SSM)

PIM-SSM is still Sparse Mode which eliminates the need for RP and RPT. With SSM, host is responsible for source discovery (typically via an out-of-band mechanism). Host and LHR need to run IGMPv3. By default you need to chose an IP in SSM range for PIM SSM group. (SSM block is 232.0.0.0 to 232.255.255.255).

