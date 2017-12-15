# Redistribution into OSPF

## Redistributing Connected Routes
* Provides and alternative to the network keyword but there are some significant differences:
  * When you use network keyword:
    * OSPF generates type-a and type-2 LSAs
    * Shows up as `O` or `O IA` routes
    * Enables OSPF on the interface
  * When you redistribute connected routes
    * Generates external route (Type-5 or Type-7 LSA)
    * Shows up as `O E1` or `O E2` routes
    * Does not enable OSPF on the interface
### Configuration Example

![ospf_r_con](https://user-images.githubusercontent.com/31813625/33814804-cd7d610a-ddfa-11e7-982f-737215031d51.png)

**R1**
<pre>
R1#show run | s ospf
router ospf 1
 network 10.10.1.1 0.0.0.0 area 0
</pre>
**ASBR**
<pre>
ASBR(config)#<b>router ospf 1</b>
ASBR(config-router)#<b>network 10.10.1.2 0.0.0.0 area 0</b>
ASBR(config-router)#<b>redistribute connected subnets</b></pre>

* `subnets` means redistribute classless subnets otherwise the router only redistributes classful subnets

Verification on R1

<pre>
R1#<b>show ip route ospf</b>
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2

Gateway of last resort is not set

      1.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
<b>O <u>E2</u>     1.1.1.0/30 [110/20] via 10.10.1.2, 00:03:06, GigabitEthernet0/0
O <u>E2</u>     1.1.2.0/24 [110/20] via 10.10.1.2, 00:03:06, GigabitEthernet0/0
O <u>E2</u>     1.3.0.0/16 [110/20] via 10.10.1.2, 00:03:06, GigabitEthernet0/0</b></pre>

<pre>
R1#<b>show ip ospf database</b>

            OSPF Router with ID (10.10.1.1) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.3.0.1         1.3.0.1         471         0x80000003 0x0045B2 1
10.10.1.1       10.10.1.1       470         0x80000002 0x00CD0C 1

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.1.1       10.10.1.1       470         0x80000001 0x00649D

		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
1.1.1.0         1.3.0.1         483         0x80000001 0x008A11 0
1.1.2.0         1.3.0.1         483         0x80000001 0x009106 0
1.3.0.0         1.3.0.1         483         0x80000001 0x008F08 0
</pre>

LSID<sub>Type-4</sub> = External Network Number. Let's open one of the AS External LSAs

<pre>
R1#<b>show ip ospf database external 1.1.1.0</b>

            OSPF Router with ID (10.10.1.1) (Process ID 1)

		Type-5 AS External Link States

  LS age: 605
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 1.1.1.0 (External Network Number )
  Advertising Router: 1.3.0.1
  LS Seq Number: 80000001
  Checksum: 0x8A11
  Length: 36
  Network Mask: /30
	Metric Type: 2 (Larger than any link state path)
	MTID: 0
	Metric: 20
	Forward Address: 0.0.0.0
	External Route Tag: 0
</pre>

If we were asked to **summarize on ASBR** those networks to 1.0.0.0/8:
<pre>
ASBR(config-router)#<b>summary-address 1.0.0.0 255.0.0.0</b></pre>
<pre>
R1#<b>show ip route ospf</b>
Gateway of last resort is not set

O E2  1.0.0.0/8 [110/20] via 10.10.1.2, 00:01:51, GigabitEthernet0/0
</pre>

## Redistributing EIGRP into OSPF
* Default metric for redistributed routes are:
  * Form BGP: Metric = 1
  * From other OSPF: The metric will be equal to the metric of the source OSPF
  * All other sources: Metric = 20
    * We can change it by subcommand `RD2(config-router)#default-metric 10
`
* <pre>redistribute <i>protocol</i> [ <i>process-id</i> | <i>as-number</i> ] [ <b>metric</b> <i>bw delay reliability load mtu</i> ] [ <b>match</b> { <b>internal</b> | <b>nssa-external</b> | <b>external 1</b> | <b>external 2</b> }] [ <b>tag</b> <i>tag-value</i> ][ <b>route-map</b> <i>name</i> ]</pre>
* By default, the OSPF redistribute command creates Type 2 routes
  * E2: The OSPF routers do not add any internal OSPF cost to the metric for an E2 route
  * E1: OSPF routers calculate the metrics of E1 routes by adding the internal cost to reach the
ASBR to the external cost defined on the redistributing ASBR

![image](https://user-images.githubusercontent.com/31813625/34020254-30a88594-e101-11e7-8376-97a2a04d4b1d.png)


Basic Configuration

**BR1**
<pre>
hostname BR1
!
interface GigabitEthernet0/0
 ip address 10.10.10.1 255.255.255.0
!
router ospf 2
 network 10.10.10.1 0.0.0.0 area 0
</pre>

**BR2**
<pre>
hostname BR2
!
interface GigabitEthernet0/0
 ip address 10.10.20.1 255.255.255.0
!
router eigrp BR2
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  network 10.10.20.1 0.0.0.0
 exit-address-family
!
</pre>

**RD1**
<pre>
hostname RD1
!
interface GigabitEthernet0/0
 ip address 10.10.10.2 255.255.255.0
!
interface GigabitEthernet0/1
 ip address 172.16.1.1 255.255.255.0
 speed 10
!
router ospf 1
 network 172.16.1.1 0.0.0.0 area 0
!
router ospf 2
 network 10.10.10.2 0.0.0.0 area 0
</pre>

**RD2**
<pre>
hostname RD2
!
interface GigabitEthernet0/0
 ip address 10.10.20.2 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 172.31.1.1 255.255.255.0
!
router eigrp RD2
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  network 10.10.20.2 0.0.0.0
 exit-address-family
!
router ospf 1
 network 172.31.1.1 0.0.0.0 area 0
</pre>

**HQ**
<pre>
hostname HQ
!
interface GigabitEthernet0/0
 ip address 172.16.1.2 255.255.255.0
 speed 10
!
interface GigabitEthernet0/1
 ip address 172.31.1.2 255.255.255.0
!
router ospf 1
 network 172.16.1.2 0.0.0.0 area 0
 network 172.31.1.2 0.0.0.0 area 0
</pre>

<pre>
HQ#<b>show ip route</b>
Gateway of last resort is not set

      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.1.0/24 is directly connected, GigabitEthernet0/0
L        172.16.1.2/32 is directly connected, GigabitEthernet0/0
      172.31.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.31.1.0/24 is directly connected, GigabitEthernet0/1
L        172.31.1.2/32 is directly connected, GigabitEthernet0/1
</pre>

### Redistribution Configuration
<pre>
RD1(config-router)#<b>redistribute ospf 2 subnets metric-type 1</b></pre>

<pre>
RD2(config-router)#<b>sredistribute eigrp 1 subnets metric-type 1</b></pre>

<pre>
HQ#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/24 is subnetted, 2 subnets
O E1     10.10.10.0 [110/<b>11</b>] via 172.16.1.1, 00:02:18, GigabitEthernet0/0
O E1     10.10.20.0 [110/<b>21</b>] via 172.31.1.1, 00:00:40, GigabitEthernet0/1
</pre>

<pre>
HQ#<b>show ip ospf neighbor</b>

Neighbor ID     Pri   State           Dead Time   Address         Interface
<b>172.31.1.1</b>        1   FULL/BDR        00:00:35    172.31.1.1      GigabitEthernet0/1
</b>172.16.1.1</b>        1   FULL/BDR        00:00:38    172.16.1.1      GigabitEthernet0/0
</pre>

<pre>
HQ#<b>show ip ospf database external adv-router 172.31.1.1</b>

            OSPF Router with ID (172.31.1.2) (Process ID 1)

		<b>Type-5 AS External Link States</b>

  LS age: 154
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 10.10.20.0 (External Network Number )
  Advertising Router: 172.31.1.1
  LS Seq Number: 80000001
  Checksum: 0x74B6
  Length: 36
  Network Mask: /24
	Metric Type: 1 (Comparable directly to link state metric)
	MTID: 0
	<b>Metric: 20</b>
	Forward Address: 0.0.0.0
	External Route Tag: 0
</pre>

<pre>
HQ#<b>show ip ospf database external adv-router 172.16.1.1</b>

            OSPF Router with ID (172.31.1.2) (Process ID 1)

		<b>Type-5 AS External Link States</b>

  LS age: 318
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 10.10.10.0 (External Network Number )
  Advertising Router: 172.16.1.1
  LS Seq Number: 80000001
  Checksum: 0x9CBA
  Length: 36
  Network Mask: /24
	Metric Type: 1 (Comparable directly to link state metric)
	MTID: 0
	<b>Metric: 1</b>
	Forward Address: 0.0.0.0
	External Route Tag: 0
</pre>

We can change the default metric `RD2(config-router)#default-metric 10`

Let's verify this change
<pre>
HQ#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/24 is subnetted, 2 subnets
O E1     10.10.10.0 [110/11] via 172.16.1.1, 00:10:19, GigabitEthernet0/0
O E1     10.10.20.0 [110/<b>11</b>] via 172.31.1.1, 00:01:57, GigabitEthernet0/1
</pre>


Now you can redistribute ospf 1 to ospf 2 and eigrp for mutual reachablity

<pre>
RD1(config)#<b>router ospf 2</b>
RD1(config-router)#redistribute ospf 1 subnets metric-type 1</b></pre>

<pre>
RD2(config)#<b>router eigrp RD2</b>
RD2(config-router)#<b>address-family ipv4 unicast autonomous-system 1</b>
RD2(config-router-af)#<b>topology base</b>
RD2(config-router-af-topology)#<b>redistribute ospf 1 metric 1000000 20 255 255 255</b>
RD2(config-router-af-topology)#<b>exit-af-topology</b></pre>

**Verification:** Look at Administrative Distances and Metrics

<pre>
BR1#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
O E1     10.10.20.0/24 [110/22] via 10.10.10.2, 00:06:03, GigabitEthernet0/0
      172.16.0.0/24 is subnetted, 1 subnets
O E1     172.16.1.0 [110/11] via 10.10.10.2, 00:06:03, GigabitEthernet0/0
      172.31.0.0/24 is subnetted, 1 subnets
O E1     172.31.1.0 [110/12] via 10.10.10.2, 00:06:03, GigabitEthernet0/0
</pre>

<pre>
BR2#<b>show ip route eigrp</b>
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
D EX     10.10.10.0/24
           [170/10250240] via 10.10.20.2, 00:04:07, GigabitEthernet0/0
      172.16.0.0/24 is subnetted, 1 subnets
D EX     172.16.1.0
           [170/10250240] via 10.10.20.2, 00:04:07, GigabitEthernet0/0
      172.31.0.0/24 is subnetted, 1 subnets
D EX     172.31.1.0
           [170/10250240] via 10.10.20.2, 00:04:07, GigabitEthernet0/0
</pre>

For `traceroute` you can issue command below to make it faster
<pre>
HQ(config)#<b>no ip icmp rate-limit unreachable</b></pre>

<pre>
HQ#<b>traceroute 10.10.10.1</b>
Type escape sequence to abort.
Tracing the route to 10.10.10.1
VRF info: (vrf in name/id, vrf out name/id)
  1 172.16.1.1 7 msec 5 msec 4 msec
  2 10.10.10.1 6 msec 7 msec 8 msec
</pre>

<pre>
BR2#<b>show ip eigrp topology 172.16.1.0/24</b>
EIGRP-IPv4 VR(BR2) Topology Entry for AS(1)/ID(10.10.20.1) for 172.16.1.0/24
  State is Passive, Query origin flag is 1, 1 Successor(s), FD is 1312030720, RIB is 10250240
  Descriptor Blocks:
  10.10.20.2 (GigabitEthernet0/0), from 10.10.20.2, Send flag is 0x0
      Composite metric is (1312030720/1311375360), route is External
      Vector metric:
        Minimum bandwidth is 1000000 Kbit
        Total delay is 20010000000 picoseconds
        Reliability is 255/255
        Load is 1/255
        Minimum MTU is 1500
        Hop count is 1
        Originating router is 172.16.1.2
      External data:
        AS number of route is 1
        <b>External protocol is OSPF, external metric is 11</b>
        Administrator tag is 0 (0x00000000)
</pre>