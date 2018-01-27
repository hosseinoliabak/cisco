# DMVPN Phase 1

![dmvpn](https://user-images.githubusercontent.com/31813625/35467591-7782b508-02de-11e8-9005-302c4dce6b2d.png)


Base configuration
<pre>
hostname HQ
!
interface GigabitEthernet0/0
 ip address 200.0.0.12 255.255.255.0
!
ip route 201.1.1.0 255.255.255.0 GigabitEthernet0/0
ip route 202.2.2.0 255.255.255.0 GigabitEthernet0/0
</pre>

<pre>
hostname Branch1
!
interface GigabitEthernet0/1
 ip address 201.1.1.1 255.255.255.0
!
no ip http server
no ip http secure-server
ip route 200.0.0.0 255.255.255.0 GigabitEthernet0/1
ip route 202.2.2.0 255.255.255.0 GigabitEthernet0/1
</pre>

<pre>
hostname Branch2
!
interface GigabitEthernet0/2
 ip address 202.2.2.2 255.255.255.0
!
no ip http server
no ip http secure-server
ip route 200.0.0.0 255.255.255.0 GigabitEthernet0/2
ip route 201.1.1.0 255.255.255.0 GigabitEthernet0/2
</pre>
<pre>
hostname Internet
!
interface GigabitEthernet0/0
 ip address 200.0.0.10 255.255.255.0
!
interface GigabitEthernet0/1
 ip address 201.1.1.10 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 202.2.2.10 255.255.255.0
</pre>

## DMVPN Phase 1 Configuration

<pre>
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>tunnel mode gre multipoint</b> 
HQ(config-if)#<b>tunnel source gigabitEthernet 0/0</b>
HQ(config-if)#<b>ip address 10.0.1.12 255.255.255.0</b>
HQ(config-if)#<b>ip nhrp map multicast dynamic</b> 
HQ(config-if)#<b>ip nhrp network-id 1</b>
</pre>
* By default, the tunnel mode is GRE point-to-point
* We don't have tunnel destination like as in GRE point-to-point. Because the tunnel destination
will be dynamic
* `ip nhrp map multicast dynamic` tells the HQ where to forward the multicast packets to.
We used this instead of using `tunnel destination` command
* `ip nhrp network-id` when you use multiple DMVPN networks this command would differenciate between them.

Configuring branches are easy as when we configure one branch we can use the configuration
as a template for other branches

<pre>
Branch1(config)#<b>interface Tunnel0</b>
Branch1(config-if)#<b>tunnel source GigabitEthernet0/1</b>
Branch1(config-if)#<b>tunnel destination 200.0.0.12</b>               
Branch1(config-if)#<b>ip address 10.0.1.1 255.255.255.0</b>
Branch1(config-if)#<b>ip nhrp map 10.0.1.12 200.0.0.12</b>   
Branch1(config-if)#<b>ip nhrp nhs 10.0.1.12</b>
Branch1(config-if)#<b>ip nhrp map multicast 200.0.0.12</b>  
Branch1(config-if)#<b>ip nhrp network-id 1</b>
</pre>
* `ip nhrp map multicast` Routing protocols such as RIP, EIGRP, and OSPF need multicast

<pre>
HQ#show dmvpn 
Legend: Attrb --> S - Static, <b>D - Dynamic</b>, I - Incomplete
	N - NATed, L - Local, X - No Socket
	T1 - Route Installed, T2 - Nexthop-override
	C - CTS Capable
	# Ent --> Number of NHRP entries with same NBMA peer
	NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
	UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel0, IPv4 NHRP Details 
Type:Hub, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 201.1.1.1              10.0.1.1    UP 00:04:33     <b>D</b>
HQ#<b>show ip nhrp</b> 
10.0.1.1/32 via 10.0.1.1
   Tunnel0 created 00:05:41, expire 01:54:18
   <b>Type: dynamic</b>, Flags: unique registered nhop 
   <b>NBMA address: 201.1.1.1</b> 

</pre>
Copy the bold lines of the output, only modify the ip address command and the source interface,
then paste it to branch2 
<pre>
Branch1#<b>show running-config interface tunnel 0</b>
Building configuration...

Current configuration : 204 bytes
!
<b>interface Tunnel0
 ip address 10.0.1.1 255.255.255.0
 ip nhrp map multicast 200.0.0.12
 ip nhrp network-id 1
 ip nhrp nhs 10.0.1.12
 tunnel source GigabitEthernet0/1
 tunnel destination 200.0.0.12
end</b></pre>

<pre>
Branch2(config)#<b>interface Tunnel0</b>
Branch2(config-if)# <b>ip address 10.0.1.2 255.255.255.0</b>
Branch2(config-if)# <b>ip nhrp map multicast 200.0.0.12</b>
Branch2(config-if)# <b>ip nhrp network-id 1</b>
Branch2(config-if)# <b>ip nhrp nhs 10.0.1.12</b>
Branch2(config-if)# <b>tunnel source GigabitEthernet0/2</b>
Branch2(config-if)# <b>tunnel destination 200.0.0.12</b>
</pre>

Verification
<pre>
HQ#<b>show dmvpn</b> 
Interface: Tunnel0, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 201.1.1.1              10.0.1.1    UP 00:29:04     D
     1 202.2.2.2              10.0.1.2    UP 00:00:08     D

HQ#<b>ping 10.0.1.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/6/10 ms
HQ#<b>ping 10.0.1.2</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/8/11 ms
</pre>

Now, Let's add LANs behind each routers and see how different protocols behaves in
DMVPN Phase 1

![dmvpnp1](https://user-images.githubusercontent.com/31813625/35468221-abd8c7d0-02e7-11e8-8fd5-231801b22396.png)


<pre>
HQ(config)#<b>interface gigabitEthernet 0/3</b>
HQ(config-if)#<b>ip address 172.16.0.12 255.255.240.0</b>
HQ(config-if)#<b>no shutdown </b></pre>
<pre>
Branch1(config)#<b>interface gigabitEthernet 0/3</b>
Branch1(config-if)#<b>ip address 192.168.1.1 255.255.255.0</b>
Branch1(config-if)#<b>no shutdown</b></pre>
<pre>
Branch2(config)#<b>interface gigabitEthernet 0/3</b>
Branch2(config-if)#<b>ip address 192.168.2.2 255.255.255.0</b>
Branch2(config-if)#<b>no shutdown</b></pre>

### RIP configuration:
<pre>
HQ(config)#<b>router rip</b> 
HQ(config-router)#<b>version 2</b>
HQ(config-router)#<b>network 10.0.1.0</b>
HQ(config-router)#<b>network 172.16.0.0</b>
HQ(config-router)#<b>no auto-summary</b>
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>no ip split-horizon</b></pre>
<pre>
Branch1(config)#<b>router rip</b> 
Branch1(config-router)#<b>version 2</b>
Branch1(config-router)#<b>network 10.0.1.0</b>
Branch1(config-router)#<b>network 192.168.1.0</b>
Branch1(config-router)#<b>no auto-summary</b></pre>
<pre>
Branch1(config)#<b>router rip</b> 
Branch1(config-router)#<b>version 2</b>
Branch2(config-router)#<b>network 10.0.1.0</b>
Branch2(config-router)#<b>network 192.168.2.0</b>
Branch2(config-router)#<b>no auto-summary</b> </pre>

Verification
<pre>
HQ#<b>show ip route rip</b>
Gateway of last resort is not set

R     192.168.1.0/24 [120/1] via 10.0.1.1, 00:00:18, Tunnel0
R     192.168.2.0/24 [120/1] via 10.0.1.2, 00:00:27, Tunnel0

HQ#<b>show dmvpn</b> 
Interface: Tunnel0, IPv4 NHRP Details 
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 201.1.1.1              10.0.1.1    UP 00:52:51     D
     1 202.2.2.2              10.0.1.2    UP 00:23:55     D
</pre>

<pre>
Branch2#<b>show ip route rip</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
R        172.16.0.0 [120/1] via 10.0.1.12, 00:00:05, Tunnel0
R     <b>192.168.1.0/24 [120/2] via <u>10.0.1.1</u>, 00:00:05, Tunnel0</b></pre>
<pre>
Branch1#<b>ping 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
Packet sent with a source address of 192.168.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/9/12 ms
Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.12 8 msec 8 msec 7 msec
  2 10.0.1.2 8 msec 8 msec 8 msec
</pre>
We can see above that everything goes through the HQ as in DMVPN phase 1, branches don't
have direct communication in between.

### EIGRP Configuration

First let's remove the RIP configuration from HQ, Branch1, and Branch2
<pre>
(config)#<b>no router rip</b></pre>
<pre>HQ(config-if)#<b>ip split-horizon</b></pre>

Now, EIGRP configuration

<pre>
Branch1(config)#<b>router eigrp 12</b>
Branch1(config-router)#<b>network 192.168.1.1 0.0.0.0</b>
Branch1(config-router)#<b>network 10.0.1.1 0.0.0.0</b>
</pre>
<pre>
Branch2(config)#<b>router eigrp 12</b>
Branch2(config-router)#<b>network 192.168.2.2 0.0.0.0</b>
Branch2(config-router)#<b>network 10.0.1.2 0.0.0.0</b> 
</pre>
<pre>
<pre>
HQ(config)#<b>router eigrp 12</b>
HQ(config-router)#<b>network 172.16.0.12 0.0.0.0</b>
HQ(config-router)#<b>network 10.0.1.12 0.0.0.0</b>
HQ(config-router)#
*Jan 27 04:09:49.545: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.0.1.2 (Tunnel0) is up: new adjacency
*Jan 27 04:09:49.547: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.0.1.1 (Tunnel0) is up: new adjacency
HQ#<b>show ip eigrp neighbors</b> 
EIGRP-IPv4 VR(HQ) Address-Family Neighbors for AS(1)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.0.1.1                Tu0                      10 00:00:46  372  2232  0  4
0   10.0.1.2                Tu0                      13 00:00:56   19  1470  0  3
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>no ip split-horizon eigrp 12</b></pre>
<pre>
Branch1#<b>show ip route eigrp</b> 
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
D        172.16.0.0 [90/26880256] via 10.0.1.12, 00:02:33, Tunnel0
D     192.168.2.0/24 [90/28160256] via 10.0.1.12, 00:01:22, Tunnel0
</pre>
<pre>
Branch2#<b>show ip route eigrp | include 192.168.1.0</b>
<b>D     192.168.1.0/24 [90/28160256] via <u>10.0.1.12</u>, 00:06:25, Tunnel0</b>
</pre>
You see that in RIP the next hop was `10.0.1.1` but EIGRP changes the next hop
when advertises networks. In this example because we are configuring DMVPN phase 1, it doesn't
matter. When we use Phase 2, this matters.

Now, let's check the path between Branch2 and Branch1
<pre>
Branch2#<b>traceroute 192.168.1.1 source 192.168.2.2</b>        
Type escape sequence to abort.
Tracing the route to 192.168.1.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.12 9 msec 8 msec 9 msec
  2 10.0.1.1 10 msec 9 msec 8 msec
</pre>

Since the traffic goes through HQ, there is no point to advertise all networks to our
branch routers. Let's configure a default route summary on the HQ router and advertise
it towards the branch routers

<pre>
HQ(config-if)#<b>ip split-horizon eigrp 12</b>
HQ(config-if)#<b>ip summary-address eigrp 12 192.168.0.0 255.255.252.0</b>
*Jan 27 05:08:15.037: %DUAL-5-NBRCHANGE: EIGRP-IPv4 12: Neighbor 10.0.1.2 (Tunnel0) is resync: summary configured
*Jan 27 05:08:15.038: %DUAL-5-NBRCHANGE: EIGRP-IPv4 12: Neighbor 10.0.1.1 (Tunnel0) is resync: summary configured
</pre>
<pre>
Branch2#<b>show ip route eigrp</b> 

Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
D        172.16.0.0 [90/26880256] via 10.0.1.12, 00:15:57, Tunnel0
D     <b>192.168.0.0/22 [90/28160256] via 10.0.1.12</b>, 00:02:56, Tunnel0
</pre>

### OSPF Configuration
* OSPF is not the best solution for DMVPN
* It is not scalable when we have dozens of routers
* Branches' routers usually don't like all the LSA flooding
  * One way to reduce the number of prefixes is to use a stub or totally stub area 
* We will try each of OSPF network types
  * broadcast
  * non-broadcast
  * point-to-point 
  * point-to-multipoint
  * point-to-multipoint non-broadcast

First let's remove the EIGRP configuration from HQ, Branch1, and Branch2
  
<pre>
HQ, Branch1, Branch2(config)#<b>no router eigrp 12</b>
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>ip summary-address eigrp 12 192.168.0.0 255.255.252.0</b></pre>
I will configure each of OSPF network types with DMVPN

#### point-to-point OSPF network type
OSPF basic configuration
Branch1
<pre>
router ospf 1
 network 10.0.1.1 0.0.0.0 area 0
 network 192.168.1.1 0.0.0.0 area 0
</pre>
Branch2
<pre>
router ospf 1
 network 10.0.1.2 0.0.0.0 area 0
 network 192.168.2.2 0.0.0.0 area 0

</pre>
HQ
<pre>
HQ(config)#<b>router ospf 1</b>
HQ(config-router)#<b>network 10.0.1.12 0.0.0.0 area 0</b>
HQ(config-router)#<b>network 172.16.0.12 0.0.0.0 area 0</b>
<b>*Jan 27 16:58:39.437: %OSPF-5-ADJCHG: Process 1, Nbr 202.2.2.2 on Tunnel0 from EXSTART to DOWN, Neighbor Down: Adjacency forced to reset
*Jan 27 16:58:39.479: %OSPF-5-ADJCHG: Process 1, Nbr 201.1.1.1 on Tunnel0 from EXSTART to DOWN, Neighbor Down: Adjacency forced to reset</b></pre>
<pre>
HQ#<b>show ip ospf interface brief</b> 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/3        1     0               172.16.0.12/20     1     DR    0/0
Tu0          1     0               10.0.1.12/24       1000  <b>P2P</b>   0/1
</pre>
The default OSPF network type for tunnel interfaces in point-to-point but we are using
point-to-multipoint interfaces n DMVPN. HQ expects one router not two. It keeps establishing
and tearing neighbor adjacencies.

<pre>
HQ#<b>show ip ospf neighbor</b> 

Neighbor ID     Pri   State           Dead Time   Address         Interface
<b>201.1.1.1</b>         0   EXSTART/  -     00:00:39    10.0.1.1        Tunnel0
HQ#<b>show ip ospf neighbor</b> 

Neighbor ID     Pri   State           Dead Time   Address         Interface
<b>202.2.2.2</b>         0   EXSTART/  -     00:00:39    10.0.1.2        Tunnel0
</pre>

So **forget about point-to-point** network type when working with DMVPN

#### broadcast OSPF network type
broadcast OSPF network type works very well with DMVPN because it establishes
neighbor adjacencies automatically

<pre>
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>ip ospf network broadcast</b>
</pre>


<pre>
Branch2(config)#<b>interface tunnel 0</b>
Branch2(config-if)#<b>ip ospf network broadcast</b>
Branch2(config-if)#<b>ip ospf priority 0</b>
</pre>
<pre>
Branch1(config)#<b>interface tunnel 0</b>
Branch1(config-if)#<b>ip ospf network broadcast</b>
Branch1(config-if)#<b>ip ospf priority 0</b></pre>
* `ip ospf priority 0`: since there is no direct communication between branches we 
don't want them to be elected as BR or BDR

<pre>
HQ#<b>show ip ospf neighbor</b> 

Neighbor ID     Pri   State           Dead Time   Address         Interface
201.1.1.1         0   FULL/DROTHER    00:00:39    10.0.1.1        Tunnel0
202.2.2.2         0   FULL/DROTHER    00:00:38    10.0.1.2        Tunnel0
</pre>
<pre>
Branch1#<b>show ip route ospf</b>

Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
O        172.16.0.0 [110/1001] via 10.0.1.12, 00:04:05, Tunnel0
O     192.168.2.0/24 [110/1001] via 10.0.1.2, 00:03:40, Tunnel0

Branch1#<b>ping 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
Packet sent with a source address of 192.168.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/9/11 ms
</pre>

#### non-broadcast OSPF network type
* non-broadcast OSPF network type works like broadcast with the exception that we have
to configure static neighbors

<pre>
HQ(config-if)#<b>ip ospf network non-broadcast</b>
HQ(config)#<b>router ospf 1</b>
HQ(config-router)#<b>neighbor 10.0.1.1</b>
HQ(config-router)#<b>neighbor 10.0.1.2</b></pre>
<pre>
Branch1 & Branch2(config-if)#<b>ip ospf network non-broadcast</b>
Branch1 & Branch2(config)#<b>router ospf 1</b>
Branch1 & Branch2(config-router)#<b>neighbor 10.0.1.12</b></pre>

<pre>
HQ#<b>show ip ospf neighbor</b>

Neighbor ID     Pri   State           Dead Time   Address         Interface
201.1.1.1         0   FULL/DROTHER    00:01:34    10.0.1.1        Tunnel0
202.2.2.2         0   FULL/DROTHER    00:01:34    10.0.1.2        Tunnel0
</pre>
<pre>
HQ#<b>show ip ospf interface brief</b> 
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/3        1     0               172.16.0.12/20     1     DR    0/0
Tu0          1     0               10.0.1.12/24       1000  DR    2/2
</pre>
<pre>
Branch1#<b>ping 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
Packet sent with a source address of 192.168.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/8/11 ms
</pre>

#### point-to-multipoint OSPF network type
point-to-multipoint OSPF network type also works very well.

Let's remove the configuration of non-broadcast network and configure the point-to-multipoint
network. We no longer need to worry about the priority of branches as there is no DR/BDR election.
we also no longer need to worry about adding adjacencies in HQ as with point-to-multipoint adjacencies formed automatically.
<pre>
HQ(config)#<b>router ospf 1</b>
HQ(config-router)#<b>no neighbor 10.0.1.1</b>
HQ(config-router)#<b>no neighbor 10.0.1.2</b>
HQ(config-if)#<b>ip ospf network point-to-multipoint</b></pre>
<pre>
Branch1(config)#<b>router ospf 1</b>
Branch1(config-router)#<b>no neighbor 10.0.1.12</b>
Branch1(config-if)#<b>ip ospf network point-to-multipoint</b></pre>
<pre>
Branch2(config)#<b>router ospf 1</b>
Branch2(config-router)#<b>no neighbor 10.0.1.12</b>
Branch2(config-if)#<b>ip ospf network point-to-multipoint </b>
*Jan 27 18:47:41.280: %OSPF-5-ADJCHG: Process 1, Nbr 200.0.0.12 on Tunnel0 from LOADING to FULL, Loading Done
</pre>
<pre>
HQ#<b>show ip ospf neighbor</b> 

Neighbor ID     Pri   State           Dead Time   Address         Interface
201.1.1.1         0   FULL/  <b>-</b>        00:01:53    10.0.1.1        Tunnel0
202.2.2.2         0   FULL/  <b>-</b>        00:01:58    10.0.1.2        Tunnel0
</pre>
* No DR/BDR election anymore
<pre>
Branch1#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
O        10.0.1.2/32 [110/2000] via 10.0.1.12, 00:05:03, Tunnel0
O        10.0.1.12/32 [110/1000] via 10.0.1.12, 00:05:48, Tunnel0
      172.16.0.0/20 is subnetted, 1 subnets
O        172.16.0.0 [110/1001] via 10.0.1.12, 00:05:48, Tunnel0
<b>O     192.168.2.0/24 [110/2001] via 10.0.1.12, 00:05:03, Tunnel0</b></pre>
As we can see Branch1 to reach Branch2, points to the HQ router which is not issue
when dealing with DMVP phase 1 is issue in phase 2.

#### point-to-multipoint non-broadcast OSPF network type
* Like point-to-multipoint OSPF network but we have to configure static neighbors
 
### BGP Configuration
I removed all ospf configuration. Let's now configure BGP

#### eBGP with different AS on branches

<pre>
HQ(config)#<b>router bgp 64500</b>
HQ(config-router)#<b>neighbor 10.0.1.1 remote-as 64501</b>
HQ(config-router)#<b>neighbor 10.0.1.2 remote-as 64502</b>
HQ(config-router)#<b>network 172.16.0.0 mask 255.255.240.0</b></pre>
<pre>
Branch1(config)#<b>router bgp 64500</b>  
Branch1(config-router)#<b>neighbor 10.0.1.12 remote-as 6500</b>  
*Jan 27 19:40:33.274: %BGP-5-ADJCHANGE: neighbor 10.0.1.12 Up 
Branch1(config-router)#<b>network 192.168.1.0 mask 255.255.255.0</b></pre>
<pre>
Branch2(config)#<b>router bgp 64502</b>
Branch2(config-router)#<b>neighbor 10.0.1.12 remote-as 64500</b>
*Jan 27 19:39:37.173: %BGP-5-ADJCHANGE: neighbor 10.0.1.12 Up 
Branch2(config-router)#<b>network 192.168.2.0 mask 255.255.255.0</b></pre>
<pre>
HQ#<b>show bgp ipv4 unicast summary</b> 

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4        64501       7       7        4    0    0 00:02:14        1
10.0.1.2        4        64502       5       9        4    0    0 00:01:09        1

HQ#<b>show bgp ipv4 unicast</b> 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.0.0/20    0.0.0.0                  0         32768 i
 *>  192.168.1.0      10.0.1.1                 0             0 64501 i
 *>  192.168.2.0      10.0.1.2                 0             0 64502 i
</pre>
<pre>
Branch1#<b>show ip route bgp</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
B        172.16.0.0 [20/0] via 10.0.1.12, 00:03:12
B     192.168.2.0/24 [20/0] via 10.0.1.2, 00:01:37
Branch1#<b>ping 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
Packet sent with a source address of 192.168.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/7/9 ms
</pre>

#### eBGP with same AS on branches
<pre>
HQ(config)#<b>router bgp 64500</b>
HQ(config-router)#<b>no neighbor 10.0.1.1 remote-as 64501</b>
HQ(config-router)#<b>no neighbor 10.0.1.2 remote-as 64502</b>
HQ(config-router)#<b>neighbor 10.0.1.1 remote-as 64512</b>
HQ(config-router)#<b>neighbor 10.0.1.2 remote-as 64512</b></pre>
<pre>
Branch1(config)#<b>router bgp 64512</b>
Branch1(config-router)#<b>neighbor 10.0.1.12 remote-as 64500</b>
Branch1(config-router)#<b>network 192.168.1.0 mask 255.255.255.0</b></pre>
<pre>
Branch2(config)#<b>router bgp 64512</b>
Branch2(config-router)#<b>neighbor 10.0.1.12 remote-as 64500</b>
Branch2(config-router)#<b>network 192.168.2.0 mask 255.255.255.0</b></pre>
As we can see below Branch1 and Branch2 don't accept routes from the other branch since they
have AS 64512 in their BGP topology AS Path.
<pre>
HQ#<b>show ip bgp neighbors 10.0.1.2 advertised-routes</b> 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.0.0/20    0.0.0.0                  0         32768 i
 *>  192.168.1.0      10.0.1.1                 0             0 64512 i
 *>  192.168.2.0      10.0.1.2                 0             0 64512 i

Total number of prefixes 3 
HQ#<b>show ip bgp neighbors 10.0.1.1 advertised-routes</b> 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.0.0/20    0.0.0.0                  0         32768 i
 *>  192.168.1.0      10.0.1.1                 0             0 64512 i
 *>  192.168.2.0      10.0.1.2                 0             0 64512 i

Total number of prefixes 3 
</pre>
<pre>
Branch1#<b>show ip route bgp</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
B        172.16.0.0 [20/0] via 10.0.1.12, 01:15:23
</pre>
<pre>
Branch2#<b>show ip route bgp</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
B        172.16.0.0 [20/0] via 10.0.1.12, 01:15:12
</pre>
But let's configure the HQ to advertise a default route to our branches
<pre>
HQ(config)#<b>ip route 0.0.0.0 0.0.0.0 null 0</b>
HQ(config)#<b>router bgp 64500</b>
HQ(config-router)#<b>network 0.0.0.0 mask 0.0.0.0</b>
</pre>
<pre>
Branch1#<b>show ip route bgp</b> 

Gateway of last resort is 10.0.1.12 to network 0.0.0.0

B*    0.0.0.0/0 [20/0] via 10.0.1.12, 00:02:20
      172.16.0.0/20 is subnetted, 1 subnets
B        172.16.0.0 [20/0] via 10.0.1.12, 01:28:57
Branch1#<b>ping 192.168.2.2 source 192.168.1.1</b></b> 
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
Packet sent with a source address of 192.168.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/9/12 ms
</pre>
<pre>
HQ#<b>show ip bgp neighbors 10.0.1.1 advertised-routes</b> 

     Network          Next Hop            Metric LocPrf Weight Path
 <b>*>  0.0.0.0          0.0.0.0                  0         32768 i</b>
 *>  172.16.0.0/20    0.0.0.0                  0         32768 i
 *>  192.168.1.0      10.0.1.1                 0             0 64512 i
 *>  192.168.2.0      10.0.1.2                 0             0 64512 i

Total number of prefixes 4 
</pre>
<pre>
Branch2#<b>show ip bgp</b> 

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          10.0.1.12                0             0 64500 i
 *>  172.16.0.0/20    10.0.1.12                0             0 64500 i
 *>  192.168.2.0      0.0.0.0                  0         32768 i
</pre>

#### iBGP with dynamic peers
* With eBGP we configured neighbors manually which defeats the purpose of having
dynamic DMVPN branch routers
* BGP supports *dynamic peers* which means HQ will accept a BGP neighbor adjacency
from any router in a given range
  * Can be used for both eBGP and iBGP but the remote routers have to be in the same AS
  * I configure here iBGP with dynamic peers

Cleanup:
<pre>
HQ(config)#<b>no router bgp 64500</b></pre>
<pre>
Branch1(config)#<b>no router bgp 64512</b></pre>
<pre>
Branch2(config)#<b>no router bgp 64512</b></pre>    

Configuration
<pre>
HQ(config)#<b>router bgp 64500</b>   
HQ(config-router)#<b>bgp listen range 10.0.1.0/24 peer-group DMVPN_BRANCHES</b>
HQ(config-router)#<b>neighbor DMVPN_BRANCHES peer-group</b> 
HQ(config-router)#<b>neighbor DMVPN_BRANCHES remote-as 64500</b>
HQ(config-router)#<b>network 0.0.0.0 mask 0.0.0.0</b>
</pre>
<pre>
Branch1(config-router)#<b>neighbor 10.0.1.12 remote-as 64500</b>
Branch1(config-router)#<b>network 192.168.1.0 mask 255.255.255.0</b>
</pre>
<pre>
Branch2(config)#<b>router bgp 64500</b>   
Branch2(config-router)#<b>neighbor 10.0.1.12 remote-as 64500</b>
Branch2(config-router)#<b>network 192.168.2.0 mask 255.255.255.0</b></pre>

<pre>
Branch1#<b>ping 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
Packet sent with a source address of 192.168.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/9/11 ms

</pre>
<pre>
Branch1#<b>show ip bgp</b>
BGP table version is 3, local router ID is 201.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 0.0.0.0          10.0.1.12                0    100      0 i
 *>  192.168.1.0      0.0.0.0                  0         32768 i

</pre>
<pre>
HQ#<b>show ip bgp summary</b> 
BGP router identifier 200.0.0.12, local AS number 64500

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
*10.0.1.1       4        64500      11      12        4    0    0 00:06:24        1
*10.0.1.2       4        64500       7       9        4    0    0 00:03:24        1
* Dynamically created based on a listen range command
Dynamically created neighbors: 2, Subnet ranges: 1

BGP peergroup DMVPN_SPOKES listen range group members: 
  10.0.1.0/24 

Total dynamically created neighbors: 2/(100 max), Subnet ranges: 1
</pre>
* Advantage of iBGP in combination with DMVPN: no need to route filtering on HQ.
Because of iBGP split-horizon, HQ won't advertise any networks from branches to the other.
<pre>
HQ#<b>show ip bgp neighbors 10.0.1.2 advertised-routes</b>    


     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          0.0.0.0                  0         32768 i

Total number of prefixes 1 
</pre>