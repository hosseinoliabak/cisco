# VXLAN Fabric Ingress Replication with BGP EVPN as Control Plane

In this post, I am going to explain how the location-identity mapping database populates using MP-BGP. Consider the image below where VTEP01 takes the original ARP L2 frame from HOST01. VTEP01 learns the IP-MAC binding of the end host. BGP Update sends the information to the Route-Reflector (iBGP), which in turn forwards this update message to all its BGP peers. With the support of BGP EVPN, the discovery of the VTEP neighbor is more dynamic.

![VXLAN-FL_Multicast drawio-1](https://user-images.githubusercontent.com/31813625/232263028-9016fcb8-8cab-42d1-93ae-4f3f9093e7f0.svg "VXLAN IR with MP-BGP as control plane")

MP-BGP in combination with Ethernet VPN (EVPN) provides the control plane aspect. EVPN does not entirely remove the need for flooding. Broadcast traffic may still incur flooding.

### VXLAN with BGP EVPN Configuration Overview

In this scenario, multidestination traffic is handled using IR (as opposed to PIM). In order to use BGP EVPN as the control plane, you need `nv overlay evpn` global configuration.

MP-BGP is known for its scalability, multitenancy, and routing policy capabilities. BGP EVPN address family (`address-family l2vpn evpn`) carries the MAC, IP, Network, VRF, and VTEP information of the hosts. Once a VTEP learns about a host behind it, BGP EVPN distributes this information to all other BGP EVPN neighbors. We have already discussed MP-BGP in this post. But I am going to give a brief summary in our BGP EVPN context.

Route Distinguisher helps us differentiate between different routes stored in the MP-BGP table. For separation and path efficiency, use a unique RD on a per-VRF, per-router basis.

Route Target – which happens to have a similar notation as RD – associates with a particular route which then allows that route to be placed in the VRF which imports that route.

* Export route-target: what routes will go from VRF into BGP
* Import route-target: what routes will go from BGP into VRF

NX-OS provides automated derivation of RDs and RTs for simplification. The snippet below is an example of the configuration in one of the leafs.


```c
LEAF01(config)# nv overlay evpn
LEAF01(config)# feature bgp
LEAF01(config)# feature vn-segment-vlan-based
LEAF01(config)# feature nv overlay
LEAF01(config)# vlan 100,200
LEAF01(config-vlan)# vlan 100
LEAF01(config-vlan)# vn-segment 20100
LEAF01(config-vlan)# vlan 200
LEAF01(config-vlan)# vn-segment 20200
LEAF01(config-vlan)# interface nve1
LEAF01(config-if-nve)# no shutdown
LEAF01(config-if-nve)# host-reachability protocol bgp
LEAF01(config-if-nve)# source-interface loopback1
LEAF01(config-if-nve)# member vni 20100
LEAF01(config-if-nve-vni)# ingress-replication protocol bgp
LEAF01(config-if-nve-vni)# member vni 20200
LEAF01(config-if-nve-vni)# ingress-replication protocol bgp
LEAF01(config-if-nve-vni)# router bgp 65000
LEAF01(config-router)# neighbor 192.168.0.1
LEAF01(config-router-neighbor)# remote-as 65000
LEAF01(config-router-neighbor)# update-source loopback0
LEAF01(config-router-neighbor)# address-family l2vpn evpn
LEAF01(config-router-neighbor-af)# send-community extended
LEAF01(config-router-neighbor-af)# neighbor 192.168.0.2
LEAF01(config-router-neighbor)# remote-as 65000
LEAF01(config-router-neighbor)# update-source loopback0
LEAF01(config-router-neighbor)# address-family l2vpn evpn
LEAF01(config-router-neighbor-af)# send-community extended
LEAF01(config-router-neighbor-af)# evpn
LEAF01(config-evpn)# vni 20100 l2
LEAF01(config-evpn-evi)# rd auto
LEAF01(config-evpn-evi)# route-target both auto
LEAF01(config-evpn-evi)# vni 20200 l2
LEAF01(config-evpn-evi)# rd auto
LEAF01(config-evpn-evi)# route-target both auto
```

VXLAN core which is unaware of the inner header (identity), forwards the VXLAN frame based on only outer header (location). So, in our case, VXLAN core – which is typically our spines, are unaware of endpoint addresses (identities). But in order to carry the MP-BGP routes, you need to enable `nv overlay evpn` on them. Spines are good choice of being BGP RRs.

```c
SPIN01(config)# nv overlay evpn
SPIN01(config)# feature bgp
SPIN01(config)# router bgp 65000
SPIN01(config-router)# template peer-policy RR_PEER_POLICY
SPIN01(config-router-ptmp)# send-community extended
SPIN01(config-router-ptmp)# route-reflector-client
SPIN01(config-router-ptmp)# template peer-session RR_PEER_SESSION
SPIN01(config-router-stmp)# remote-as 65000
SPIN01(config-router-stmp)# update-source loopback0
SPIN01(config-router-stmp)# neighbor 192.168.1.1
SPIN01(config-router-neighbor)# inherit peer-session RR_PEER_SESSION
SPIN01(config-router-neighbor)# address-family l2vpn evpn
SPIN01(config-router-neighbor-af)# inherit peer-policy RR_PEER_POLICY 1
SPIN01(config-router-neighbor-af)# neighbor 192.168.1.2
SPIN01(config-router-neighbor)# inherit peer-session RR_PEER_SESSION
SPIN01(config-router-neighbor)# address-family l2vpn evpn
SPIN01(config-router-neighbor-af)# inherit peer-policy RR_PEER_POLICY 1
SPIN01(config-router-neighbor-af)# neighbor 192.168.1.3
SPIN01(config-router-neighbor)# inherit peer-session RR_PEER_SESSION
SPIN01(config-router-neighbor)# address-family l2vpn evpn
SPIN01(config-router-neighbor-af)# inherit peer-policy RR_PEER_POLICY 1
SPIN01(config-router-neighbor-af)# neighbor 192.168.1.4
SPIN01(config-router-neighbor)# inherit peer-session RR_PEER_SESSION
SPIN01(config-router-neighbor)# address-family l2vpn evpn
SPIN01(config-router-neighbor-af)# inherit peer-policy RR_PEER_POLICY 1
```
VTEP02 and VTEP03 receive the VXLAN frame and then perform deencapsulation. They both will learn MAC address of HOST01 and they map it to VTEP01.

There are two other notes here to mention: the subsequent messages are unicast if VTEP01 finds HOST02 location and identity. If there were a way which could distribute the same location-identity mapping across all edges, then the need for flooding could be eliminated.

The last note before we jump into the workshop is that we will discuss the verification commands along with BGP EVPN Route types in the next post.

## Workshop

In this scenario, we will configure our very first VXLAN topology with Flood and Learn as well as with Ingress Replication for BUM traffic. In this scenario, IS-IS is configured for underlay:

![VXLAN Fabric Ingress Replication with BGP EVPN as Control Plane Workshop](https://user-images.githubusercontent.com/31813625/232261114-774992f1-bed8-4042-b95a-5de440b84077.jpg "VXLAN Fabric IR with MP-BGP Control Plane Workshop")

<details>
 
<summary>SPINE01</summary>

```elixir
nv overlay evpn
feature bgp
feature isis

interface Ethernet1/1
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/3
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/4
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface loopback0
  ip address 192.168.0.1/32
  ip router isis UNDERLAY
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.0001.00
  is-type level-2
router bgp 65000
  template peer-policy RR_PEER_POLICY
    send-community extended
    route-reflector-client
  template peer-session RR_PEER_SESSION
    remote-as 65000
    update-source loopback0
  neighbor 192.168.1.1
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.1.2
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.1.3
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.1.4
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
```
</details>

<details>

<summary>SPINE02</summary>

```elixir
nv overlay evpn
feature bgp
feature isis

interface Ethernet1/1
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/3
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/4
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface loopback0
  ip address 192.168.0.2/32
  ip router isis UNDERLAY


router isis UNDERLAY
  net 49.0000.0000.0002.00
  is-type level-2
router bgp 65000
  template peer-policy RR_PEER_POLICY
    send-community extended
    route-reflector-client
  template peer-session RR_PEER_SESSION
    remote-as 65000
    update-source loopback0
  neighbor 192.168.1.1
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.1.2
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.1.3
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.1.4
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
```
</details>

<details>

<summary>LEAF01</summary>
 
```elixir
nv overlay evpn
feature bgp
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    ingress-replication protocol bgp
  member vni 20200
    ingress-replication protocol bgp

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.1/32
  ip router isis UNDERLAY

interface loopback1
  description NVE
  ip address 192.168.250.1/32
  ip router isis UNDERLAY
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1001.00
  is-type level-2

router bgp 65000
  template peer-policy RR_PEER_POLICY
    send-community extended
  neighbor 192.168.0.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
  neighbor 192.168.0.2
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
evpn
  vni 20100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 20200 l2
    rd auto
    route-target import auto
    route-target export auto
```

</details>

<details>

<summary>LEAF02</summary>

```elixir
nv overlay evpn
feature bgp
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    ingress-replication protocol bgp
  member vni 20200
    ingress-replication protocol bgp

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.2/32
  ip router isis UNDERLAY

interface loopback1
  description NVE
  ip address 192.168.250.2/32
  ip router isis UNDERLAY
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1002.00
  is-type level-2

router bgp 65000
  template peer-policy RR_PEER_POLICY
    send-community extended
  neighbor 192.168.0.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
  neighbor 192.168.0.2
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
evpn
  vni 20100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 20200 l2
    rd auto
    route-target import auto
    route-target export auto
```
</details>

<details>

<summary>LEAF03</summary>

```elixir
nv overlay evpn
feature bgp
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    ingress-replication protocol bgp
  member vni 20200
    ingress-replication protocol bgp

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/41
  switchport access vlan 200
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.3/32
  ip router isis UNDERLAY

interface loopback1
  description NVE
  ip address 192.168.250.3/32
  ip router isis UNDERLAY
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1003.00
  is-type level-2

router bgp 65000
  template peer-policy RR_PEER_POLICY
    send-community extended
  neighbor 192.168.0.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
  neighbor 192.168.0.2
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
evpn
  vni 20100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 20200 l2
    rd auto
    route-target import auto
    route-target export auto
```
</details>

<details>

<summary>LEAF04</summary>

```elixir
nv overlay evpn
feature bgp
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    ingress-replication protocol bgp
  member vni 20200
    ingress-replication protocol bgp

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/41
  switchport access vlan 200
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.4/32
  ip router isis UNDERLAY

interface loopback1
  description NVE
  ip address 192.168.250.4/32
  ip router isis UNDERLAY
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1004.00
  is-type level-2

router bgp 65000
  template peer-policy RR_PEER_POLICY
    send-community extended
  neighbor 192.168.0.1
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
  neighbor 192.168.0.2
    remote-as 65000
    update-source loopback0
    address-family l2vpn evpn
      send-community extended
evpn
  vni 20100 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 20200 l2
    rd auto
    route-target import auto
    route-target export auto
```
</details>