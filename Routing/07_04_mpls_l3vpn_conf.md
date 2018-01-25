# Configuring MPLS L3VPN

## Configuring VRFs
<pre>
PE2(config)#<b>ip vrf CUSTOMER-A</b>
PE2(config-vrf)#<b>rd 2345:1</b>     
PE2(config-vrf)#<b>route-target import 2345:1</b>
PE2(config-vrf)#<b>route-target export 2345:1</b>
PE2(config-vrf)#<b>ip vrf CUSTOMER-B</b>
PE2(config-vrf)#<b>rd 2345:2</b>                 
PE2(config-vrf)#<b>route-target import 2345:2</b>
PE2(config-vrf)#<b>route-target export 2345:2</b>
PE2(config-vrf)#<b>do show ip vrf</b>
  Name                             Default RD            Interfaces
  CUSTOMER-A                       2345:1                
  CUSTOMER-B                       2345:2
PE2(config)#<b>int gigabitEthernet 0/1.12</b>
PE2(config-subif)#<b>ip vrf forwarding CUSTOMER-A</b>
PE2(config-subif)#<b>encapsulation dot1Q 12</b>
PE2(config-subif)#<b>ip address 192.168.12.2 255.255.255.0</b>
PE2(config-subif)#<b>int gigabitEthernet 0/0.22</b>
PE2(config-subif)#<b>ip vrf forwarding CUSTOMER-B</b>
PE2(config-subif)#<b>encapsulation dot1Q 22</b>               
PE2(config-subif)#<b>ip address 192.168.22.2 255.255.255.0</b>
PE2(config-subif)#<b>do sho ip vrf</b>
  Name                             Default RD            Interfaces
  CUSTOMER-A                       2345:1                Gi0/1.12
  CUSTOMER-B                       2345:2                Gi0/0.22
PE1#<b>ping vrf CUSTOMER-A 192.168.12.1</b>
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/7/9 ms
PE1#<b>show ip route vrf CUSTOMER-A</b>

Routing Table: CUSTOMER-A
Gateway of last resort is not set

      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, GigabitEthernet0/1.12
L        192.168.12.2/32 is directly connected, GigabitEthernet0/1.12
</pre>
You have to configure PE2 as well. I don't put the configuration here. But at the end, I will put
`show run` of all devices.

## Configuring MP-iBGP
<pre>
PE1(config)#<b>ip bgp new-format</b> 
PE1(config)#<b>router bgp 2345</b>
PE1(config-router)#<b>no bgp default ipv4-unicast</b> 
PE1(config-router)#<b>neighbor 5.5.5.5 remote-as 2345</b>
PE1(config-router)#<b>neighbor 5.5.5.5 update-source loopback 0</b>
PE1(config-router)#<b>address-family vpnv4</b>
PE1(config-router-af)#<b>neighbor 5.5.5.5 activate</b> 
PE1(config-router-af)#<b>neighbor 5.5.5.5 send-community both</b>
</pre>
<pre>
PE2(config)#<b>ip bgp new-format</b> 
PE2(config)#<b>router bgp 2345</b>
PE2(config-router)#<b>no bgp default ipv4-unicast</b>
PE2(config-router)#<b>neighbor 2.2.2.2 remote-as 2345</b>
PE2(config-router)#<b>neighbor 2.2.2.2 update-source loopback 0</b>
PE1(config-router)#<b>address-family vpnv4</b>
PE2(config-router-af)#<b>neighbor 2.2.2.2 act</b>
PE2(config-router-af)#<b>neighbor 2.2.2.2 send-community both</b>
</pre>

<pre>
PE1#<b>show ip bgp vpnv4 all summary</b> 
BGP router identifier 2.2.2.2, local AS number 2345
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
5.5.5.5         4         2345       4       2        1    0    0 00:00:07        0
</pre>

<pre>
PE2#<b>show ip bgp vpnv4 all summary</b>
BGP router identifier 5.5.5.5, local AS number 2345
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2.2.2.2         4         2345       2       4        1    0    0 00:00:11        0
</pre>

## PE-CE routing
* At this stage we are not yet learning any customer routing information
and we are not injecting that into BGP so that those routes can be shared between
our PEs
* We are going to configure BGP in PE-CE for CUSTOMER-A
* We are going to configure EIGRP in PE-CE for CUSTOMER-B then we have 
to perform mutual route redistribution
* At the end of this section we have a fully functional MPLS L3VPN

### PE-CE BGP routing for CUSTOMER-A
<pre>
A-CE1(config)#<b>route-map CONNECTED permit 10</b>
A-CE1(config-route-map)#<b>match interface loopback 0</b>
A-CE1(config)#<b>router bgp 65012</b>
A-CE1(config-router)#<b>redistribute connected route-map CONNECTED</b>
A-CE1(config-router)#<b>neighbor 192.168.12.2 send-community both</b> 
</pre>
<pre>
PE1(config)#<b>router bgp 2345</b>
PE1(config-router)#<b>neighbor 192.168.12.1 remote-as 65012</b>
PE1(config-router)#<b>address-family ipv4 vrf CUSTOMER-A</b>
PE1(config-router-af)#<b>neighbor 192.168.12.1 remote-as 65012</b>
PE1(config-router-af)#<b>neighbor 192.168.12.1 activate</b> 
PE1(config-router-af)#<b>neighbor 192.168.12.1 send-community both</b>
PE1(config-router-af)#<b>redistribute connected</b> 
</pre>
<pre>
A-CE1#<b>show ip bgp summary</b> 
BGP router identifier 1.1.1.1, local AS number 65012
BGP table version is 3, main routing table version 3
2 network entries using 288 bytes of memory
2 path entries using 160 bytes of memory
2/2 BGP path/bestpath attribute entries using 304 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
1 BGP extended community entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 800 total bytes of memory
BGP activity 2/0 prefixes, 2/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.12.2    4         2345       6       6        3    0    0 00:01:13        1
A-CE1#<b>show ip bgp</b>         
     Network          Next Hop            Metric LocPrf Weight Path
 *>  1.1.1.1/32       0.0.0.0                  0         32768 ?
 r>  192.168.12.0     192.168.12.2             0             0 2345 ?
</pre>
Exercise: Do something similar for PE2 and A-CE2.

Verification command after fully connected site 1 and site 2 of CUSTOMER-A
<pre>
PE1#<b>show bgp vpnv4 unicast all summary</b> 
BGP router identifier 2.2.2.2, local AS number 2345
BGP table version is 8, main routing table version 8
4 network entries using 624 bytes of memory
4 path entries using 320 bytes of memory
5/4 BGP path/bestpath attribute entries using 800 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
1 BGP extended community entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1816 total bytes of memory
BGP activity 4/0 prefixes, 4/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
5.5.5.5         4         2345      66      65        8    0    0 00:56:03        2
192.168.12.1    4        65012      14      17        8    0    0 00:09:07        1
</pre>
<pre>
A-CE1#<b>show ip route bgp</b> 
Gateway of last resort is not set

      6.0.0.0/32 is subnetted, 1 subnets
B        6.6.6.6 [20/0] via 192.168.12.2, 00:03:30
B     192.168.56.0/24 [20/0] via 192.168.12.2, 00:03:08
A-CE1#<b>ping 6.6.6.6 source loopback 0</b>
Packet sent with a source address of 1.1.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/24/46 ms
</pre>

### PE-CE EIGRP routing for CUSTOMER-B
<pre>
B-SW-CE2(config)#<b>router eigrp 1</b>
B-SW-CE2(config-router)#<b>network 22.22.22.22 0.0.0.0</b>
B-SW-CE2(config-router)#<b>network 192.168.22.22 0.0.0.0</b>
</pre>
<pre>
PE1(config)#<b>router eigrp 65535</b> # This ASN doesn't really matter.
PE1(config-router)#<b>address-family ipv4 vrf CUSTOMER-B</b>
PE1(config-router-af)#<b>autonomous-system 1</b> 
PE1(config-router-af)#<b>no auto-summary</b> 
PE1(config-router-af)#<b>network 192.168.22.2 0.0.0.0</b>
*Jan 24 22:45:04.202: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.22.22 (GigabitEthernet0/0.22) is up: new adjacency
PE1(config-router-af)#<b>default-metric 10000000 10 255 1 1500</b>
PE1(config-router-af)#<b>redistribute bgp 2345</b>
PE1(config)#<b>router bgp 2345</b>
PE1(config-router)#<b>address-family ipv4 vrf CUSTOMER-B</b>
PE1(config-router-af)#<b>redistribute connected</b> 
PE1(config-router-af)#<b>redistribute eigrp 1</b>  
</pre>
Exercise: Do something similar for PE2 and B-SW-CE1.
Then verify
<pre>
PE1#<b>show ip eigrp vrf CUSTOMER-B neighbors</b> 
EIGRP-IPv4 Neighbors for AS(1) VRF(CUSTOMER-B)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   192.168.22.22           Gi0/0.22                 13 00:10:39   20   120  0  3
</pre>

### Verification of our MPLS L3VPN and analysis
<pre>
A-CE2#<b>show bgp ipv4 unicast</b> 
BGP table version is 7, local router ID is 6.6.6.6
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  1.1.1.1/32       192.168.56.5                           0 2345 65012 ?
 *>  6.6.6.6/32       0.0.0.0                  0         32768 ?
 *>  192.168.12.0     192.168.56.5                           0 2345 ?
 r>  192.168.56.0     192.168.56.5             0             0 2345 ?
</pre>
<pre>
A-CE2#<b>show bgp ipv4 unicast summary</b> 

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.56.5    4         2345      38      35        7    0    0 00:27:18        3
</pre>

<pre>
PE2#<b>show bgp vpnv4 unicast vrf CUSTOMER-A summary</b>

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.56.6    4        65056      36      40       14    0    0 00:28:21        1
</pre>

<pre>
PE2#<b>show bgp vpnv4 unicast vrf CUSTOMER-A</b>

     Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 2345:1 (default for vrf CUSTOMER-A)
 *>i 1.1.1.1/32       2.2.2.2                  0    100      0 65012 ?
 *>  6.6.6.6/32       192.168.56.6             0             0 65056 ?
 *>i 192.168.12.0     2.2.2.2                  0    100      0 ?
 *>  192.168.56.0     0.0.0.0                  0         32768 ?
</pre>

<pre>
PE2#<b>show bgp vpnv4 unicast vrf CUSTOMER-A 6.6.6.6</b>
BGP routing table entry for <b>2345:1:6.6.6.6/32</b>, version 7
Paths: (1 available, best #1, table CUSTOMER-A)
  Advertised to update-groups:
     6         
  Refresh Epoch 1
  65056
    192.168.56.6 (via vrf CUSTOMER-A) from 192.168.56.6 (6.6.6.6)
      Origin incomplete, metric 0, localpref 100, valid, external, best
      <b>Extended Community: RT:2345:1</b>
      <b>mpls labels in/out 505/nolabel</b>
      rx pathid: 0, tx pathid: 0x0

</pre>
* `2345:1:6.6.6.6/32` is the RD
    <pre>
    PE2#<b>show ip vrf</b> 
      Name                             Default RD            Interfaces
      <b>CUSTOMER-A                       2345:1                Gi0/1.56</b>
      CUSTOMER-B                       2345:2                Gi0/0.11
    </pre>
* `Extended Community: RT:2345:1` is the RT
* `mpls labels in/out 505/nolabel`:
  * In MPLS we have 1 label applied to all routers in MPLS network.
  * In MPLS VPN we have 2 labels:
    * Inner label (VNPv4 label): As PE2 takes this route (6.6.6.6/32) into the BGP
    table, it generates this MPLS label which we use later.
    * The outer label:This is which is used by every router in the MPLS network
<pre>
PE1#<b>show bgp vpnv4 unicast vrf CUSTOMER-A</b>
BGP table version is 14, local router ID is 2.2.2.2

     Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: <b>2345:1</b> (default for vrf CUSTOMER-A)
 *>  1.1.1.1/32       192.168.12.1             0             0 65012 ?
 <b>*>i 6.6.6.6/32       5.5.5.5                  0    100      0 65056 ?</b>
 *>  192.168.12.0     0.0.0.0                  0         32768 ?
 *>i 192.168.56.0     5.5.5.5                  0    100      0 ?
</pre> 
Let's look at FIB table for CUSTOMER-A on PE1
<pre>
PE1#<b>show ip cef vrf CUSTOMER-A</b>
Prefix               Next Hop             Interface
0.0.0.0/0            no route
0.0.0.0/8            drop
0.0.0.0/32           receive              
1.1.1.1/32           192.168.12.1         GigabitEthernet0/1.12
<b>6.6.6.6/32           10.0.23.3            GigabitEthernet0/3.23</b>
127.0.0.0/8          drop
192.168.12.0/24      attached             GigabitEthernet0/1.12
192.168.12.0/32      receive              GigabitEthernet0/1.12
192.168.12.1/32      attached             GigabitEthernet0/1.12
192.168.12.2/32      receive              GigabitEthernet0/1.12
192.168.12.255/32    receive              GigabitEthernet0/1.12
192.168.56.0/24      10.0.23.3            GigabitEthernet0/3.23
224.0.0.0/4          drop
224.0.0.0/24         receive              
240.0.0.0/4          drop
255.255.255.255/32   receive    
</pre> 
<pre>
PE1#<b>show ip cef vrf CUSTOMER-A 6.6.6.6</b>
6.6.6.6/32
  nexthop 10.0.23.3 GigabitEthernet0/3.23 label <b>303() 505()</b>
</pre> 
* 303: outer label
* 505: inner label (VPNv4 label)
* Let's see what LSR1 does with this packet. as we can see it forwards towards LSR2.
    <pre>
    LSR1#show mpls forwarding-table 
    Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
    Label      Label      or Tunnel Id     Switched      interface              
    300        Pop Label  4.4.4.4/32       0             Gi0/0.34   10.0.34.4   
    301        Pop Label  2.2.2.2/32       35494         Gi0/2.23   10.0.23.2   
    302        Pop Label  10.0.45.0/24     232           Gi0/0.34   10.0.34.4   
    <b>303        402        5.5.5.5/32       38113         Gi0/0.34   10.0.34.4</b></pre>
    As we can see then LSR2 pops off the outer label (402) then forwards the packet to PE2
    <pre>
    LSR2#sho mpls forwarding-table 
    Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
    Label      Label      or Tunnel Id     Switched      interface              
    400        Pop Label  3.3.3.3/32       0             Gi0/0.34   10.0.34.3   
    401        Pop Label  10.0.23.0/24     0             Gi0/0.34   10.0.34.3   
    <b>402        Pop Label  5.5.5.5/32       36153         Gi0/2.45   10.0.45.5</b>   
    403        301        2.2.2.2/32       38184         Gi0/0.34   10.0.34.3 
    </pre>
    In PE2 then the inner label 505 and sends just the plain IP packet over the router A-CE2.
    <pre>
    PE2#show mpls forwarding-table 
    Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
    Label      Label      or Tunnel Id     Switched      interface              
    500        Pop Label  4.4.4.4/32       0             Gi0/3.45   10.0.45.4   
    501        400        3.3.3.3/32       0             Gi0/3.45   10.0.45.4   
    502        Pop Label  10.0.34.0/24     0             Gi0/3.45   10.0.45.4   
    503        401        10.0.23.0/24     0             Gi0/3.45   10.0.45.4   
    504        403        2.2.2.2/32       0             Gi0/3.45   10.0.45.4   
    <b>505        No Label   6.6.6.6/32[V]    590           Gi0/1.56   192.168.56.6</b>
    506        No Label   192.168.56.0/24[V] 0             aggregate/CUSTOMER-A 
    507        No Label   11.11.11.11/32[V]  590           Gi0/0.11   192.168.11.11
    508        No Label   192.168.11.0/24[V] 0             aggregate/CUSTOMER-B 
    </pre>