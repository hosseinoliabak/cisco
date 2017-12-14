# EIGRP Stub Configuration

![eigrp_stub](https://user-images.githubusercontent.com/31813625/33972008-85ed1f0a-e049-11e7-93bf-d37ad7d8713d.png)

Basic Router and EIGRP Configuration
**R1**
<pre>
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.0
!
interface Loopback1
 ip address 11.11.11.11 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.12.1 255.255.255.252
!
router eigrp R1
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
   redistribute connected
  exit-af-topology
  network 192.168.12.1 0.0.0.0
 exit-address-family
</pre>
**R2**
<pre>
hostname R2
!
interface GigabitEthernet0/1
 ip address 192.168.12.2 255.255.255.252
!
interface GigabitEthernet0/3
 ip address 192.168.23.2 255.255.255.248
!
interface GigabitEthernet0/4
 ip address 192.168.24.2 255.255.255.248
!
router eigrp R2
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  network 192.168.12.2 0.0.0.0
  network 192.168.23.2 0.0.0.0
  network 192.168.24.2 0.0.0.0
 exit-address-family
</pre>
**R3**
<pre>
!
hostname R3
!
interface Loopback0
 ip address 10.10.3.3 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.23.3 255.255.255.248
!
router eigrp R3
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  network 10.10.3.3 0.0.0.0
  network 192.168.23.3 0.0.0.0
 exit-address-family
</pre>
**R4**
<pre>
hostname R4
!
interface Loopback0
 ip address 10.10.4.4 255.255.255.0
!
interface GigabitEthernet0/2
 ip address 192.168.24.4 255.255.255.248
!
router eigrp R4
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  network 10.10.4.4 0.0.0.0
  network 192.168.24.4 0.0.0.0
 exit-address-family
</pre>

Detailed Neighbor's information on R2:
<pre>
R2#<b>show ip eigrp neighbors detail</b>
EIGRP-IPv4 VR(R2) Address-Family Neighbors for AS(1)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   192.168.12.1            Gi0/1                    10 00:00:38 1604  5000  0  17
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 2
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

2   192.168.24.4            Gi0/4                    11 00:26:23   27   162  0  11
   Time since Restart 00:06:24
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 1
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

1   192.168.23.3            Gi0/3                    10 00:27:02   22   132  0  12
   Time since Restart 00:06:24
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 1
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

Max Nbrs: 0, Current Nbrs: 0
</pre>
Now I am going to make R1 as an `eigrp stub connected summary`
<pre>
R1(config-router-af)#<b>eigrp stub</b>
*Dec 14 02:30:33.757: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (GigabitEthernet0/2) is down: peer info changed
*Dec 14 02:30:34.455: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (GigabitEthernet0/2) is up: new adjacency
</pre>
Now Let's see the result on R2
<pre>
R2#<b>show ip eigrp neighbors detail</b>
EIGRP-IPv4 VR(R2) Address-Family Neighbors for AS(1)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
<b>0   192.168.12.1            Gi0/1</b>                    10 00:01:44 1602  5000  0  19
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 2
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

   <b>Stub Peer Advertising (CONNECTED SUMMARY ) Routes
   Suppressing queries</b>
2   192.168.24.4            Gi0/4                    14 00:28:58   25   150  0  13
   Time since Restart 00:08:59
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 1
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

1   192.168.23.3            Gi0/3                    13 00:29:37   22   132  0  14
   Time since Restart 00:08:59
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 1
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

Max Nbrs: 0, Current Nbrs: 0
</pre>
Let's see the routing table on R3
<pre>
R3#show ip route eigrp
Gateway of last resort is not set

      1.0.0.0/24 is subnetted, 1 subnets
<b>D EX     1.1.1.0 [170/16000] via 192.168.23.2, 00:03:36, GigabitEthernet0/2</b>
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
D        10.10.4.0/24
           [90/16000] via 192.168.23.2, 00:30:57, GigabitEthernet0/2
      11.0.0.0/24 is subnetted, 1 subnets
<b>D EX     11.11.11.0 [170/16000] via 192.168.23.2, 00:03:36, GigabitEthernet0/2</b>
      192.168.12.0/30 is subnetted, 1 subnets
D        192.168.12.0
           [90/15360] via 192.168.23.2, 00:10:55, GigabitEthernet0/2
      192.168.24.0/29 is subnetted, 1 subnets
D        192.168.24.0
           [90/15360] via 192.168.23.2, 00:10:55, GigabitEthernet0/2
</pre>
We can still see the redistributed route while the R1 is `stub connected`  not `stub redistributed`. Why?
Because connected redistributed in considered as connected.

Now I am going to make R1 as an `eigrp stub receive-only`
<pre>
R1(config-router-af)#<b>eigrp stub receive-only</b>
*Dec 14 02:34:33.874: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (GigabitEthernet0/2) is down: peer info changed
*Dec 14 02:34:37.520: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.2 (GigabitEthernet0/2) is up: new adjacency
</pre>
Now let's see the results in R2 and R3
<pre>
R2#<b>show ip eigrp neighbors detail</b>
EIGRP-IPv4 VR(R2) Address-Family Neighbors for AS(1)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
<b>0   192.168.12.1            Gi0/1</b>                    13 00:03:44    3   100  0  21
   Version 20.0/2.0, Retrans: 0, Retries: 0
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

   <b>Receive-Only Peer Advertising (No) Routes
   Suppressing queries</b>
2   192.168.24.4            Gi0/4                    12 00:35:00   24   144  0  14
   Time since Restart 00:15:01
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 1
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

1   192.168.23.3            Gi0/3                    12 00:35:39   20   120  0  15
   Time since Restart 00:15:01
   Version 20.0/2.0, Retrans: 1, Retries: 0, Prefixes: 1
   Topology-ids from peer - 0
   Topologies advertised to peer:   base

Max Nbrs: 0, Current Nbrs: 0
</pre>

<pre>
R3#<b>show ip route eigrp</b>
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
D        10.10.4.0/24
           [90/16000] via 192.168.23.2, 00:31:26, GigabitEthernet0/2
      192.168.12.0/30 is subnetted, 1 subnets
D        192.168.12.0
           [90/15360] via 192.168.23.2, 00:11:24, GigabitEthernet0/2
      192.168.24.0/29 is subnetted, 1 subnets
D        192.168.24.0
           [90/15360] via 192.168.23.2, 00:11:24, GigabitEthernet0/2
</pre>