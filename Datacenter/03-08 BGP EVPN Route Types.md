# BGP EVPN Route Types
From the previous posts, we learned that in order to differentiate between routes in the MP-BGP table, we include a 64-bit RD value along with the MAC address. In this post, we will discuss what else other than MAC address can propagate through BGP EVPN.

Different route types carry BGP EVPN information. RFC 7432 defines route types 0 to 4; then with later drafts, they defined other route types. For us, in CCIE Datacenter, knowing route types 2, 3, and 5 are more important.

### BGP EVPN Type 2 Route – MAC/IP Advertisement Route
The main driver of running VXLAN is advertising MAC addresses to other VTEPs over layer 3. That could be done in a pure data plane with flood and learn or by using BGP EVPN. BGP EVPN advertises those MAC addresses using Route Type 2.

Route Type 2 has a mandatory MAC address and MAC address Length field. We can also advertise host IPs (/32 or /128) using route type 2.


![image-3](https://user-images.githubusercontent.com/31813625/232264437-8e5a2abd-759d-4f52-ac6b-887929b0f465.png)
*BGP EVPN Type 2 Route – MAC/IP Advertisement Route*

Once a VTEP learns MAC/IP information of an end host, it sends a BGP update including route type 2 to other VTEPs. The total length of route type 2 can be:

  * 216 bits: MAC-Only
  * 272 bits: MAC + IPv4 + L3VNI
  * 386 bits: MAC + IPv6 + L3VNI

The example below shows BGP EVPN Route Type 2 (MAC only):
```c
LEAF01(config)# show bgp l2vpn evpn 5000.000e.0000
BGP routing table information for VRF default, address family L2VPN EVPN

Route Distinguisher: 192.168.1.4:32967
BGP routing table entry for [2]:[0]:[0]:[48]:[5000.000e.0000]:[0]:[0.0.0.0]/216,
Route_Type_2:ESI:ETI:MAC_Address_Length:MAC_Addr:IP_Address_Length:IP_Address:/Whole_Length
 version 335
Paths: (2 available, best #2)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW

  Path type: internal, path is valid, not best reason: Neighbor Address, no labeled nexthop
  AS-Path: NONE, path sourced internal to AS
    192.168.250.4 (metric 81) from 192.168.0.2 (192.168.0.2)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 20200
      Extcommunity: RT:65000:20200 ENCAP:8
      Originator: 192.168.1.4 Cluster list: 192.168.0.2

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop
             Imported to 1 destination(s)
             Imported paths list: L2-20200
  AS-Path: NONE, path sourced internal to AS
    192.168.250.4 (metric 81) from 192.168.0.1 (192.168.0.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 20200 (L2VNI)
      Extcommunity: RT:65000:20200 ENCAP:8 (Overlay Encapsulation 8 means VXLAN)
      Originator: 192.168.1.4 Cluster list: 192.168.0.1

  Path-id 1 not advertised to any peer
``` 
The example below shows BGP EVPN Route Type 2 (MAC + IPv4 + L3VNI):

```c
LEAF01(config)# show bgp l2vpn evpn 192.168.200.102
BGP routing table information for VRF default, address family L2VPN EVPN
Route Distinguisher: 192.168.1.2:32967
BGP routing table entry for [2]:[0]:[0]:[48]:[5000.000c.0000]:[32]:[192.168.200.102]/272, version 319
Paths: (2 available, best #1)
Flags: (0x000202) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not in HW

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop
             Imported to 3 destination(s)
             Imported paths list: SMENODE L3-23967 L2-20200
  AS-Path: NONE, path sourced internal to AS
    192.168.250.2 (metric 81) from 192.168.0.1 (192.168.0.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 20200 23967
      Extcommunity: RT:65000:20200 RT:65000:23967 ENCAP:8 Router MAC:5002.0000.1b08
      Originator: 192.168.1.2 Cluster list: 192.168.0.1

  Path type: internal, path is valid, not best reason: Neighbor Address, no labeled nexthop
  AS-Path: NONE, path sourced internal to AS
    192.168.250.2 (metric 81) from 192.168.0.2 (192.168.0.2)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 20200 23967
      Extcommunity: RT:65000:20200 RT:65000:23967 ENCAP:8 Router MAC:5002.0000.1b08
      Originator: 192.168.1.2 Cluster list: 192.168.0.2

  Path-id 1 not advertised to any peer
```
### BGP EVPN Type 3 Route – Inclusive Multicast Ethernet Tag Route

As soon as we configure our NVE interface using ingress replication on a VTEP, it propagates Type 3 Routes to other VTEPs.

![type3](https://user-images.githubusercontent.com/31813625/232264563-e17ca501-94c3-4531-b1c8-f14097308f1a.png)
*BGP EVPN Type 3 Route – Inclusive Multicast Ethernet Tag Route*

For example, consider the configuration on the left, `ingress-replication proto bgp` propagates the other peers’ information which reveals which VNI is behind which VTEP:

<table>
<tr>
<th>IR BGP</th>
<th>FL Multicast</th>
</tr>
<tr>
<td>

```c
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    ingress-replication proto bgp
  member vni 20200
    ingress-replication proto bgp
```

</td>
<td>

```c
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200

```

</td>
</tr>
</table>

The example below shows BGP EVPN Route Type 3
```c
LEAF03(config)# show bgp l2vpn evpn 192.168.250.2
BGP routing table information for VRF default, address family L2VPN EVPN
  route Distinguisher: 192.168.1.3:32867    (L2VNI 20100)
BGP routing table entry for [3]:[0]:[32]:[192.168.250.2]/88, version 548
Paths: (1 available, best #1)
Flags: (0x000012) (high32 00000000) on xmit-list, is in l2rib/evpn, is not in HW

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop
             Imported from 192.168.1.2:32867:[3]:[0]:[32]:[192.168.250.2]/88
  AS-Path: NONE, path sourced internal to AS
    192.168.250.2 (metric 81) from 192.168.0.1 (192.168.0.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Extcommunity: RT:65000:20100 ENCAP:8
      Originator: 192.168.1.2 Cluster list: 192.168.0.1
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 20100, Tunnel Id: 192.168.250.2

  Path-id 1 not advertised to any peer

Route Distinguisher: 192.168.1.3:32967    (L2VNI 20200)
BGP routing table entry for [3]:[0]:[32]:[192.168.250.2]/88, version 555
Paths: (1 available, best #1)
Flags: (0x000012) (high32 00000000) on xmit-list, is in l2rib/evpn, is not in HW

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop
             Imported from 192.168.1.2:32967:[3]:[0]:[32]:[192.168.250.2]/88
  AS-Path: NONE, path sourced internal to AS
    192.168.250.2 (metric 81) from 192.168.0.1 (192.168.0.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Extcommunity: RT:65000:20200 ENCAP:8
      Originator: 192.168.1.2 Cluster list: 192.168.0.1
      PMSI Tunnel Attribute:
        flags: 0x00, Tunnel type: Ingress Replication
        Label: 20200, Tunnel Id: 192.168.250.2

  Path-id 1 not advertised to any peer
```
### BGP EVPN Type 5 Route – IP prefix route
BGP EVPN also provides IP prefix-based routing. Route type 5 is similar to route type 2 (/32 or /128 host route) but you can advertise the IP prefix route. Once a VTP receives the route type 5 NLRI, it imports the IP prefix route into the routing table. These are the use cases of route type 5:
  * External routing

![type5](https://user-images.githubusercontent.com/31813625/232264564-a1f8c8d6-5c5c-4410-9eec-6aaa9b0bd55a.png)
*BGP EVPN Type 5 Route – IP prefix route*
