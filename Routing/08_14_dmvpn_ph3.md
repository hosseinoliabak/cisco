# DMVPN Phase 3

![dmvpnp1](https://user-images.githubusercontent.com/31813625/35468221-abd8c7d0-02e7-11e8-8fd5-231801b22396.png)

Here are the configuration of DMVPN phase 2 on *tunnel 0*

HQ
<pre>
interface Tunnel0
 ip address 10.0.1.12 255.255.255.0
 no ip redirects
 ip nhrp map multicast dynamic
 ip nhrp network-id 1
 tunnel source GigabitEthernet0/0
 tunnel mode gre multipoint
</pre>
Branch1:
<pre>
interface Tunnel0
 ip address 10.0.1.1 255.255.255.0
 no ip redirects
 ip nhrp map multicast 200.0.0.12
 ip nhrp map 10.0.1.12 200.0.0.12
 ip nhrp network-id 1
 ip nhrp nhs 10.0.1.12
 tunnel source GigabitEthernet0/1
 tunnel mode gre multipoint
</pre>
Branch2
<pre>
interface Tunnel0
 ip address 10.0.1.1 255.255.255.0
 no ip redirects
 ip nhrp map multicast 200.0.0.12
 ip nhrp map 10.0.1.12 200.0.0.12
 ip nhrp network-id 1
 ip nhrp nhs 10.0.1.12
 tunnel source GigabitEthernet0/1
 tunnel mode gre multipoint
</pre>

## DMVPN Phase 2 Configuration

To migrate form phase 2 to phase 3 we only need two more tunnel interface subcommands
* `ip nhrp redirect` on HQ: will inform the branches that they can reach another branch
directly
* `ip nhrp shortcut` on Branches: to make changes in CEF entry when they receive a redirect message from the hub 

<pre>
HQ(config)#<b>interface tunnel 0</b>
HQ(config-if)#<b>ip nhrp redirect</b></pre>
<pre>
Spoke1 & Spoke2(config)#<b>interface tunnel 0</b>
Spoke1 & Spoke2(config-if)#<b>ip nhrp shortcut</b></pre>
Unlike the DMVPN phase 2, branches don't need specific entries in their routing table.

### RIP configuration
<pre>
HQ(config)#<b>router rip</b> 
HQ(config-router)#<b>version 2</b> 
HQ(config-router)#<b>network 10.0.1.12</b> 
HQ(config-router)#<b>default-information originate</b></pre>
<pre>
Branch1(config)#<b>router rip</b>
Branch1(config-router)#<b>version 2</b>
Branch1(config-router)#<b>network 10.0.1.1</b>
Branch1(config-router)#<b>network 192.168.1.1</b></pre>
<pre>
Branch2(config)#<b>router rip</b>
Branch2(config-router)#<b>version 2</b>
Branch2(config-router)#<b>network 10.0.1.2</b>
Branch2(config-router)#<b>network 192.168.2.2</b></pre>

<pre>
Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 8 msec 9 msec 11 msec

Branch1#<b>show dmvpn</b> 
Legend: Attrb --> <b>S - Static, D - Dynamic</b>, I - Incomplete
	<b>T1 - Route Installed</b>, T2 - Nexthop-override
==========================================================================
Interface: Tunnel0, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 202.2.2.2              10.0.1.2    UP 00:10:28   DT1
                              10.0.1.2    UP 00:10:28   DT1
     1 200.0.0.12            10.0.1.12    UP 02:47:22     S
</pre>

### EIGRP configuration:
* I already removed all RIP configuration
* EIGRP on DMVPN phase 3 works very well
* No need to disable split horizon since the spoke routers don't have to learn each others network
<pre>
HQ(config)#<b>router eigrp 120</b>
HQ(config-router)#<b>network 10.0.1.12 0.0.0.0</b></pre>
HQ(config)#interface tunnel 0
HQ(config-if)#ip summary-address eigrp 120 0.0.0.0 0.0.0.0

<pre>
Branch1(config)#<b>router eigrp 120</b>
Branch1(config-router)#<b>network 10.0.1.1 0.0.0.0</b>
Branch1(config-router)#<b>network 192.168.1.1 0.0.0.0</b></pre>
<pre>
Branch2(config)#<b>router eigrp 120</b>
Branch2(config-router)#<b>network 10.0.1.2 0.0.0.0</b>
Branch2(config-router)#<b>network 192.168.2.2 0.0.0.0</b></pre>

<pre>
HQ#<b>show ip route eigrp</b> 
Gateway of last resort is 0.0.0.0 to network 0.0.0.0

D*    0.0.0.0/0 is a summary, 00:00:33, Null0
D     192.168.1.0/24 [90/26880256] via 10.0.1.1, 00:04:31, Tunnel0
D     192.168.2.0/24 [90/26880256] via 10.0.1.2, 00:03:53, Tunnel0
</pre>
<pre>
Branch1#<b>show ip route eigrp</b> 
Gateway of last resort is 10.0.1.12 to network 0.0.0.0

D*    0.0.0.0/0 [90/28160000] via 10.0.1.12, 00:00:41, Tunnel0

Branch1#show dmvpn 
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
	T1 - Route Installed, T2 - Nexthop-override
==========================================================================

Interface: Tunnel0, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 202.2.2.2              10.0.1.2    UP 00:25:35   DT1
                              10.0.1.2    UP 00:25:35   DT1
     1 200.0.0.12            10.0.1.12    UP 03:02:29     S

Branch1#<b>show ip route nhrp</b> 
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP

Gateway of last resort is 10.0.1.12 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
H        10.0.1.2/32 is directly connected, 00:26:27, Tunnel0
H     192.168.2.0/24 [250/255] via 10.0.1.2, 00:03:11, Tunnel0
</pre>

### OSPF configuration
* I already remove EIGRP configuration
* OSPF is not best choice for DMVPN phase 3

#### broadcast OSPF network type
* Will work

<pre>
HQ & Spokes(config)#<b>interface tunnel 0</b>
HQ & Spokes(config-if)#<b>ip ospf network broadcast</b>
Branch1 & Branch2(config-if)#<b>ip ospf priority 0</b></pre>
<pre>
HQ(config)#<b>router ospf 1</b>
HQ(config-router)#<b>network 172.16.0.12 0.0.0.0 area 0</b>
HQ(config-router)#<b>network 10.0.1.12 0.0.0.0 area 1</b></pre>
<pre>
Branch1(config)#<b>router ospf 1</b>
Branch1(config-router)#<b>network 10.0.1.1 0.0.0.0 area 1</b>
Branch1(config-router)#<b>network 192.168.1.1 0.0.0.0 area 1</b></pre>
<pre>
Branch2(config)#<b>router ospf 1</b>
Branch2(config-router)#<b>network 10.0.1.2 0.0.0.0 area 1</b>
Branch2(config-router)#<b>network 192.168.2.2 0.0.0.0 area 1</b></pre>

<pre>
HQ#<b>show ip route ospf</b>
Gateway of last resort is not set

O     192.168.1.0/24 [110/1001] via 10.0.1.1, 00:02:24, Tunnel0
O     192.168.2.0/24 [110/1001] via 10.0.1.2, 00:01:35, Tunnel0
</pre>

<pre>
Branch1#<b>show ip route ospf</b>
Gateway of last resort is not set

      172.16.0.0/20 is subnetted, 1 subnets
O IA     172.16.0.0 [110/1001] via 10.0.1.12, 00:02:42, Tunnel0
O     192.168.2.0/24 [110/1001] via 10.0.1.2, 00:01:43, Tunnel0

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 9 msec 7 msec 9 msec
</pre>
#### point-to-multipoint OSPF network type
* In DMVPN phase 2 we couldn't use this type of network
* In DMVPN phase 3 if we have to configure OSPF, this network type is the best choice
since you don't have to worry about DR/BDR election
<pre>
HQ & Spokes(config)#<b>interface tunnel 0</b></pre>

<pre>
HQ#<b>show ip ospf neighbor</b> 

Neighbor ID     Pri   State           Dead Time   Address         Interface
202.2.2.2         0   FULL/  -        00:01:55    10.0.1.2        Tunnel0
201.1.1.1         0   FULL/  -        00:01:56    10.0.1.1        Tunnel0
</pre>

<pre>
Branch1#<b>show ip route ospf</b>
       + - replicated route, <b>% - next hop override</b>, p - overrides from PfR
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
<b>O   %    10.0.1.2/32 [110/2000] via 10.0.1.12</b>, 00:00:35, Tunnel0
O        10.0.1.12/32 [110/1000] via 10.0.1.12, 00:00:45, Tunnel0
      172.16.0.0/20 is subnetted, 1 subnets
O IA     172.16.0.0 [110/1001] via 10.0.1.12, 00:00:45, Tunnel0
O     192.168.2.0/24 [110/2001] via 10.0.1.12, 00:00:35, Tunnel0

Branch1#<b>show ip cef 192.168.2.2</b>
192.168.2.0/24
  <b>nexthop 10.0.1.12 Tunnel0</b>

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.12 8 msec 8 msec 9 msec
  2 10.0.1.2 15 msec 38 msec 12 msec
</pre>
First traceroute went from the HQ, but in the second try traffic went directly to Branch2 
<pre>  
Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 8 msec 7 msec 7 msec

Branch1#<b>show ip cef 192.168.2.2</b>
192.168.2.0/24
  <b>nexthop 10.0.1.2 Tunnel0</b>
  
Branch1#<b>show dmvpn</b>
T1 - Route Installed, <b>T2 - Nexthop-override</b>                           
Interface: Tunnel0, IPv4 NHRP Details 
Type:Spoke, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     2 <b>202.2.2.2              10.0.1.2    UP 00:01:03</b>   D<b>T2</b>
                              10.0.1.2    UP 00:01:03   DT2
     1 200.0.0.12            10.0.1.12    UP 00:04:03     S
</pre>

### BGP Configuration
* I removed all configuration related to OSPF

#### iBGP with dynamic peers
<pre>
HQ(config)#ip route 0.0.0.0 0.0.0.0 null 0
HQ(config)#router bgp 64500   
HQ(config-router)#bgp listen range 10.0.1.0/24 peer-group DMVPN_BRANCHES
HQ(config-router)#network 0.0.0.0 
HQ(config-router)#neighbor DMVPN_BRANCHES peer-group 
HQ(config-router)#neighbor DMVPN_BRANCHES remote-as 64500
HQ(config-router)#neighbor DMVPN_BRANCHES route-map BRANCH_ROUTERS out
</pre>