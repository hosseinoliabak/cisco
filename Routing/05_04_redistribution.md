# Redistribution into EIGRP


![redistributeintoeigrp](https://user-images.githubusercontent.com/31813625/35128613-90ff59e6-fc85-11e7-8dfc-5b2918b1f0e8.png)

R1 Configuration
<pre>
hostname R1
!
interface GigabitEthernet0/1
 ip address 10.10.10.1 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 10.10.20.1 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
router ospf 1
 network 10.10.10.1 0.0.0.0 area 0
 network 10.10.20.1 0.0.0.0 area 0
</pre>
RD1 Configuration
<pre>
hostname RD1
!
interface GigabitEthernet0/0
 ip address 10.10.11.1 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.10.2 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
router eigrp RD1
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   redistribute ospf 1 metric 1000000 20 255 1 1500
  exit-af-topology
  network 10.10.11.1 0.0.0.0
 exit-address-family
!
router ospf 1
 redistribute eigrp 1 subnets
 network 10.10.10.2 0.0.0.0 area 0
</pre>
RD2 Configuration
<pre>
hostname RD2
!
interface GigabitEthernet0/0
 ip address 10.10.21.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.20.2 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
router eigrp RD2
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   redistribute ospf 1 metric 1000000 20 255 1 1500
  exit-af-topology
  network 10.10.21.1 0.0.0.0
 exit-address-family
!
router ospf 1
 redistribute eigrp 1 subnets
 network 10.10.20.2 0.0.0.0 area 0
</pre>
RD3 Configuration
<pre>
hostname RD3
!
interface GigabitEthernet0/0
 ip address 172.20.0.1 255.255.0.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.11.2 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 10.10.21.2 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
router eigrp RD3
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   redistribute rip metric 1000000 2000 255 1 1500
  exit-af-topology
  network 10.10.11.2 0.0.0.0
  network 10.10.21.2 0.0.0.0
 exit-address-family
 !
 service-family ipv4 autonomous-system 1
  !
  topology base
  exit-sf-topology
 exit-service-family
!
router rip
 version 2
 redistribute eigrp 1 metric 2
 network 172.20.0.0
 no auto-summary
!
</pre>
R2 Configuration
<pre>
hostname R2
!
interface GigabitEthernet0/0
 ip address 172.20.0.255 255.255.0.0
 duplex auto
 speed auto
 media-type rj45
!
router rip
 version 2
 network 172.20.0.0
 no auto-summary
!
</pre>
Let's verify the end to end reachability
<pre>
R1#<b>traceroute 172.20.0.255</b>
  1 10.10.10.2 5 msec 6 msec 4 msec
  2 10.10.11.2 8 msec 7 msec 5 msec
  3 172.20.0.255 9 msec 13 msec 9 msec
</pre>
We see that the connectivity between R1 and R2 passes from RD1. 
Every thing seems to be okay and we have end-to-end reachability! But there is a problem.
We have inefficient routing.
Let's check how RD2 reaches to 172.20.0.0
<pre>
RD2#<b>traceroute 172.20.0.255</b>
  1 10.10.20.1 7 msec 6 msec 22 msec
  2 10.10.10.2 10 msec 9 msec 9 msec
  3 10.10.11.2 10 msec 9 msec 8 msec
  4 172.20.0.255 13 msec 9 msec 8 msec
</pre>
We see that if RD2 wants to reach R2 - instead of using RD3 - it goes to
R1, RD1, then RD3 which is sub-optimal!!

Here is the reason:
* When you redistribute RIP into EIGRP on RD3, both RD1 and RD2 sees the
172.16.0.0 having AD=170 if we didn't have OSPF. But here we are having
OSPF and EIGRP mutual redistribution. The router which first started the
redistribution process (RD1) is in privilege position. RD2 is going to get
RIP from OSPF because it offers better AD=110 than AD=170.
<pre>
RD1#<b>show ip route 172.20.0.0</b>
Routing entry for 172.20.0.0/16
  <b>Known via "eigrp 1", distance 170</b>, metric 10250240, type external
  Redistributing via ospf 1, eigrp 1
  Advertised by ospf 1 subnets
  Last update from 10.10.11.2 on GigabitEthernet0/0, 00:12:30 ago
  Routing Descriptor Blocks:
  * 10.10.11.2, from 10.10.11.2, 00:12:30 ago, via GigabitEthernet0/0
      Route metric is 10250240, traffic share count is 1
      Total delay is 20010 microseconds, minimum bandwidth is 1000000 Kbit
      Reliability 255/255, minimum MTU 1500 bytes
      Loading 1/255, Hops 1
RD2#<b>show ip route 172.20.0.0</b>
Routing entry for 172.20.0.0/16
  <b>Known via "ospf 1", distance 110</b>, metric 20, type extern 2, forward metric 2
  Redistributing via eigrp 1
  Advertised by eigrp 1 metric 1000000 20 255 1 1500
  Last update from 10.10.20.1 on GigabitEthernet0/1, 00:12:19 ago
  Routing Descriptor Blocks:
  * 10.10.20.1, <b>from 10.10.11.1</b>, 00:12:19 ago, via GigabitEthernet0/1
      Route metric is 20, traffic share count is 1
</pre>

### Here are the solutions:
1. Use per-route Administrative Distance (AD)
2. Use route map to filter the subnet
  * Both RD1 and RD2 apply a route map to their redistribution from OSPF
  into EIGRP, filtering routes with prefix 172.20.0.0
3. Use route tags to prevent domain loops
  * A route tag is a unitless 32-bit integer. Most routing protocols can
  assign to any given route. It is like sticky note on a specific route
<pre>
RD1(config-router)#router ospf 1
RD1(config-router)#redistribute eigrp 1 subnets route-map TAG11
RD1(config)#route-map TAG11 permit 10
RD1(config-route-map)#set tag 11

RD1(config)#router eigrp RD1
RD1(config-router)#a ipv4 a 1
RD1(config-router-af)#topology base
RD1(config-router-af-topology)#redistribute ospf 1 metric 1000000 20 255 1 1500 route-map STOPTAG12
RD1(config)#route-map STOPTAG12 deny 10
RD1(config-route-map)#match tag 12
RD1(config)#route-map STOPTAG12 permit 20

</pre>
<pre>
RD2(config)#router eigrp RD2
RD2(config-router)#a ipv4 a 1
RD2(config-router-af)#topology base 
RD2(config-router-af-topology)#redistribute ospf 1 metric 1000000 20 255 1 1500 route-map STOPTAG11        
RD2(config)#route-map STOPTAG11 deny 10
RD2(config-route-map)#match tag 11
RD2(config)#route-map STOPTAG11 permit 20

RD2(config)#router ospf 1
RD2(config-router)#redistribute eigrp 1 subnets route-map TAG12
RD2(config)#route-map TAG12 permit 10
RD2(config-route-map)#set tag 12
</pre>    