# DMVPN Phase 2
* Now branches will use multipoint GRE interface instead of point-to-point GRE interfaces
* Hence we don't configure manual destination on HQ

![dmvpnp1](https://user-images.githubusercontent.com/31813625/35468221-abd8c7d0-02e7-11e8-8fd5-231801b22396.png)

The following was phase 1 configuration

HQ: DMVPN phase 1 tunnel 0 configuration
<pre>
interface Tunnel0
 ip address 10.0.1.12 255.255.255.0
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
</pre>
Branch1: DMVPN phase 1 tunnel 0 configuration
<pre>
interface Tunnel0
 ip address 10.0.1.1 255.255.255.0
 ip nhrp map multicast 200.0.0.12
 ip nhrp map 10.0.1.12 200.0.0.12
 ip nhrp network-id 1
 ip nhrp nhs 10.0.1.12
 tunnel source GigabitEthernet0/1
 tunnel destination 200.0.0.12
</pre>
Branch2: DMVPN phase 1 tunnel 0 configuration
<pre>
interface Tunnel0
 ip address 10.0.1.2 255.255.255.0
 ip nhrp map multicast 200.0.0.12
 ip nhrp map 10.0.1.12 200.0.0.12
 ip nhrp network-id 1
 ip nhrp nhs 10.0.1.12
 tunnel source GigabitEthernet0/2
 tunnel destination 200.0.0.12
</pre>

## DMVPN Phase 2 Configuration

HQ's configuration remains the same. No changes on the HQ compared to phase 1
<pre>
HQ(config)#<b>no ip route 0.0.0.0 0.0.0.0 Null0</b> # Was here from BGP configuration
</pre>
Branch1:
<pre>
Branch1(config-if)#<b>no tunnel destination 200.0.0.12</b>
Branch1(config-if)#<b>tunnel mode gre multipoint</b> 
</pre>
<pre>
Branch2(config-if)#<b>no tunnel destination 200.0.0.12</b>
Branch2(config-if)#<b>tunnel mode gre multipoint</b> 
</pre>
<pre>
HQ#<b>show dmvpn</b>
Legend: Attrb --> S - Static, <b>D - Dynamic</b>, I - Incomplete

Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 201.1.1.1              10.0.1.1    UP 06:10:16     <b>D</b>
     1 202.2.2.2              10.0.1.2    UP 06:10:16     <b>D</b>
</pre>
<pre>
Branch1#<b>show dmvpn | begin Spoke</b>
Type:Spoke, NHRP Peers:1, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 200.0.0.12            10.0.1.12    UP 00:03:34     S
</pre>

### RIP configuration:
<pre>
HQ(config)#<b>router rip</b>
HQ(config-router)#<b>version 2</b>
HQ(config-router)#<b>no auto-summary</b>
HQ(config-router)#<b>network 172.16.0.12</b>
HQ(config-router)#<b>network 10.0.1.12</b>
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>no ip split-horizon</b> 

</pre>
<pre>
Branch1(config)#<b>router rip</b>
Branch1(config-router)#<b>version 2</b>
Branch1(config-router)#<b>no auto-summary</b>
Branch1(config-router)#<b>network 192.168.1.1</b>
Branch1(config-router)#<b>network 10.0.1.1</b>
</pre>
<pre>
Branch2(config)#<b>router rip</b>
Branch2(config-router)#<b>version 2</b>
Branch2(config-router)#<b>no auto-summary</b>
Branch2(config-router)#<b>network 192.168.2.2</b>
Branch2(config-router)#<b>network 10.0.1.2</b>
</pre>
<pre>
Branch1#<b>show ip route rip</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
R        172.16.0.0 [120/1] via 10.0.1.12, 00:00:03, Tunnel0
R     <b>192.168.2.0/24 [120/2] via 10.0.1.2</b>, 00:00:03, Tunnel0

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  <b>1 10.0.1.2 9 msec 6 msec 10 msec</b></pre>
As we can see, we have branch to branch directly route.
The traffic doesn't go through the HQ.

<pre>
Branch1#<b>show dmvpn</b>
Legend: Attrb --> <b>S - Static, D - Dynamic</b>, I - Incomplete
==========================================================================

Interface: Tunnel0, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     <b>1 202.2.2.2              10.0.1.2    UP 00:17:54     D</b>
     1 200.0.0.12            10.0.1.12    UP 00:22:07     S
</pre>

### EIGRP configuration:
<pre>
HQ(config)#<b>no router rip</b>
HQ(config-if)#<b>no ip split-horizon eigrp 120</b></pre>
<pre>
Branch1 & Branch2(config)#<b>no router rip</b></pre>
<pre>
HQ(config)#<b>router eigrp 120</b>
HQ(config-router)#<b>network 172.16.0.1 0.0.0.0</b>
HQ(config-router)#<b>network 10.0.1.12 0.0.0.0</b></pre>
<pre>
Branch1(config-router)#<b>network 192.168.1.1 0.0.0.0</b>
Branch1(config-router)#<b>network 10.0.1.1 0.0.0.0</b>   
Branch1(config-router)#
*Jan 27 23:50:51.688: %DUAL-5-NBRCHANGE: EIGRP-IPv4 120: Neighbor 10.0.1.12 (Tunnel0) is up: new adjacency
</pre>
<pre>
Branch2(config-router)#<b>network 192.168.2.2 0.0.0.0</b>
Branch2(config-router)#<b>network 10.0.1.2 0.0.0.0</b>   
Branch2(config-router)#
*Jan 27 23:49:36.908: %DUAL-5-NBRCHANGE: EIGRP-IPv4 120: Neighbor 10.0.1.12 (Tunnel0) is up: new adjacency
</pre>

<pre>
HQ#<b>show  ip eigrp neighbors</b> 
EIGRP-IPv4 Neighbors for AS(120)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.0.1.2                Tu0                      10 00:00:25   13  1470  0  2
0   10.0.1.1                Tu0                      10 00:01:10   15  1470  0  2
</pre>

<pre>
Branch1#<b>show ip route eigrp</b> 
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
D        172.16.0.0 [90/26880256] via 10.0.1.12, 00:00:48, Tunnel0
D     <b>192.168.2.0/24 [90/28160256] via 10.0.1.12</b>, 00:00:48, Tunnel0

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  <b>1 10.0.1.12 7 msec 7 msec 6 msec
  2 10.0.1.2 6 msec 9 msec 7 msec</b></pre>
* You see that this is not what we want because Spoke1 takes HQ ro reach Spoke2. Because,
EIGRP updated the next hop IP address to itself.
* To solve this:
<pre>
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>no ip next-hop-self eigrp 120</b></pre>
<pre>
Branch1#<b>show ip route eigrp</b> 
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
D        172.16.0.0 [90/26880256] via 10.0.1.12, 00:00:09, Tunnel0
D     192.168.2.0/24 [90/28160256] via 10.0.1.2, 00:00:09, Tunnel0

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 9 msec 7 msec 9 msec
</pre>

### OSPF configuration:
<pre>
HQ(config)#<b>no router eigrp 120</b>
Branch1(config)#<b>no router eigrp 120</b>
Branch2(config)#<b>no router eigrp 120</b></pre>

#### point-to-point OSPF network type
* We discussed why poin-to-point network type doesn't work with DMVPN as th HQ
expects one neighbor on its tunnels interface while there are two here.

#### broadcast OSPF network type
* Best choice for DMVPN phase 2
<pre>
HQ, Branch1, and Branch2(config)#<b>interface tunnel 0</b>
HQ, Branch1, and Branch2(config-if)#<b>ip ospf network broadcast</b></pre>

<pre>
Branch1, Branch2(config-if)#<b>ip ospf priority 0</b></pre>
Best practice ospf area number for DMVPN:
<pre>
HQ(config)#<b>router ospf 1</b>
HQ(config-router)#<b>network 172.16.0.12 0.0.0.0 area 0</b>
HQ(config-router)#<b>network 10.0.1.12 0.0.0.0 area 1</b> 
</pre>
<pre>
Branch1(config)#<b>router ospf 1</b>
Branch1(config-router)#<b>network 192.168.1.1 0.0.0.0 area 1</b>
Branch1(config-router)#<b>network 10.0.1.1 0.0.0.0 area 1</b> 
</pre>
<pre>
Branch2(config)#<b>router ospf 1</b>
Branch2(config-router)#<b>network 192.168.2.2 0.0.0.0 area 1</b>
Branch2(config-router)#<b>network 10.0.1.2 0.0.0.0 area 1</b> 
</pre>
<pre>
HQ#<b>show ip ospf neighbor</b>

Neighbor ID     Pri   State           Dead Time   Address         Interface
201.1.1.1         0   FULL/DROTHER    00:00:35    10.0.1.1        Tunnel0
202.2.2.2         0   FULL/DROTHER    00:00:35    10.0.1.2        Tunnel0
</pre>
<pre>
Branch1#<b>show ip route ospf</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
O IA     172.16.0.0 [110/1001] via 10.0.1.12, 00:01:42, Tunnel0
O     192.168.2.0/24 [110/1001] via 10.0.1.2, 00:00:51, Tunnel0

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 8 msec 7 msec 8 msec
</pre>

#### non-broadcast OSPF network type
The end result will be the same as *broadcast*. The only difference is the manual neighbor commands.

#### point-to-multipoint OSPF network type
In DMVPN phase 2 avoid this network type because all traffics go through HQ

#### point-to-multipoint non-broadcast OSPF network type
point-to-multipoint non-broadcast has the same issue as the previous one
(point-to-multipoint)

### BGP Configuration
* I removed all OSPF configuration from previous configuration
* With eBGP we cannot have same AS on the branches since they wouldn't accept prefixes
that have the same AS number in the AS bath. There are still some dirty tricks like AS override

#### eBGP with different AS on branches
<pre>
HQ(config)#<b>router bgp 64512</b>
HQ(config-router)#<b>neighbor 10.0.1.1 remote-as 64501</b>
HQ(config-router)#<b>neighbor 10.0.1.2 remote-as 64502</b>
HQ(config-router)#<b>network 172.16.0.0 mask 255.255.255.240</b>
</pre>
<pre>
Branch1(config)#<b>router bgp 64501</b>
Branch1(config-router)#<b>neighbor 10.0.1.12 remote-as 64512</b>
Branch1(config-router)#<b>network 192.168.1.0 mask 255.255.255.0</b>
</pre>
<pre>
Branch2(config)#<b>router bgp 64502</b>
Branch2(config-router)#<b>neighbor 10.0.1.12 remote-as 64512</b>
Branch2(config-router)#<b>network 192.168.2.0 mask 255.255.255.0</b>
</pre>

<pre>
HQ#<b>show ip route bgp | b Gateway</b> 
Gateway of last resort is not set

B     192.168.1.0/24 [20/0] via 10.0.1.1, 00:01:48
B     192.168.2.0/24 [20/0] via 10.0.1.2, 00:00:21
</pre>
<pre>
Branch1#<b>show ip route bgp</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
B        172.16.0.0 [20/0] via 10.0.1.12, 00:02:06
B     192.168.2.0/24 [20/0] via 10.0.1.2, 00:02:48

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 9 msec 9 msec 7 msec
</pre>

#### iBGP with dynamic peers
<pre>
HQ(config-router)#<b>no router bgp 64512</b>
Branch1(config)#<b>no router bgp 64501</b>
Branch2(config-router)#<b>no router bgp 64502</b></pre>

<pre>
HQ(config)#<b>router bgp 64500</b>
HQ(config-router)#<b>bgp listen range 10.0.1.0/24 peer-group DMVPN_BRANCHES</b>
HQ(config-router)#<b>neighbor DMVPN_BRANCHES peer-group</b>
HQ(config-router)#<b>neighbor DMVPN_BRANCHES remote-as 64500</b>
HQ(config-router)#<b>neighbor DMVPN_BRANCHES route-reflector-client</b> 
HQ(config-router)#<b>network 172.16.0.0 mask 255.255.240.0</b></pre>
<pre>
Branch1(config)#<b>router bgp 64500</b>
Branch1(config-router)#<b>neighbor 10.0.1.12 remote-as 64500</b>
Branch1(config-router)#<b>network 192.168.1.0 mask 255.255.255.0</b></pre>
<pre>
Branch2(config)#<b>router bgp 64500</b>
Branch2(config-router)#<b>neighbor 10.0.1.12 remote-as 64500</b>
Branch2(config-router)#<b>network 192.168.2.0 mask 255.255.255.0</b></pre>

<pre>
HQ#<b>show ip route bgp</b>
Gateway of last resort is not set

B     192.168.1.0/24 [200/0] via 10.0.1.1, 00:01:08
B     192.168.2.0/24 [200/0] via 10.0.1.2, 00:00:19
</pre>

<pre>
Branch1#<b>show ip route bgp</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
B        172.16.0.0 [200/0] via 10.0.1.12, 00:01:40
B     192.168.2.0/24 [200/0] via 10.0.1.2, 00:00:39

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 8 msec 6 msec 7 msec
</pre>

