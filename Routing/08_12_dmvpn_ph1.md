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
* `ip nhrp map multicast dynamic` tells the hub where to forward the multicast packets to.
We used this instead of using `tunnel destination` command
* `ip nhrp network-id` when you use multiple DMVPN networks this command would differenciate between them.

Configuring spokes are easy as when we configure one spoke we can use the configuration
as a template for other spokes

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
HQ#ping 10.0.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/8/11 ms
</pre>

Now, Let's add LANs behind each routers and see how different protocols behaves in
DMVPN Phase 1

![dmvpnp1](https://user-images.githubusercontent.com/31813625/35468221-abd8c7d0-02e7-11e8-8fd5-231801b22396.png)


<pre>
HQ(config)#interface gigabitEthernet 0/3
HQ(config-if)#ip address 172.16.0.12 255.255.240.0
HQ(config-if)#no shutdown 
</pre>
<pre>
Branch1(config)#interface gigabitEthernet 0/3
Branch1(config-if)#ip address 192.168.1.1 255.255.255.0
Branch1(config-if)#no shutdown 
</pre>
<pre>
Branch2(config)#interface gigabitEthernet 0/3
Branch2(config-if)#ip address 192.168.2.2 255.255.255.0
Branch2(config-if)#no shutdown 
</pre>

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
We can see above that everything goes through the hub as in DMVPN phase 1, spokes don't
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
HQ#show ip eigrp neighbors 
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

Now, let's chack the path beteen Spoke2 and Spoke1
<pre>
Branch2#<b>traceroute 192.168.1.1 source 192.168.2.2</b>        
Type escape sequence to abort.
Tracing the route to 192.168.1.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.12 9 msec 8 msec 9 msec
  2 10.0.1.1 10 msec 9 msec 8 msec
</pre>

Since the traffic goes through hub, there is no point to advertise all networks to our
spoke routers. Let's configure a default route summary on the hub router and advertise
it towards the spoke routers

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

