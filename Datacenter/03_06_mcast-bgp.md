# VXLAN Fabric Multicast with MP-BGP as Control Plane

In this section, we are going to discuss MP-BGP VXLAN with PIM to handle multi-destination traffic. So, your complete bipartite graph must support PIM. This is why in [this post](https://github.com/hosseinoliabak/cisco/blob/master/Datacenter/03_02_underlay.md), we happened to choose loopback 254 on the spines for our PIM Anycast RP.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232263601-6aff1eb2-048b-4104-ac3e-014383cb079b.svg" alt="VXLAN MP-BGP EVPN with Multicast in the fabric">
  <figcaption>VXLAN MP-BGP EVPN with Multicast in the fabric</figcaption>
</figure>

VTEP V1 then sends out the packet toward the IP core. The multicast tree forwards the packet until it reaches all interested receivers.

VTEP V3 receives and then de-encapsulates the VXLAN packet; because VTEP V3 has VNI 20100 and is part of subscribers for Multicast Group 239.1.1.2.

### VXLAN with BGP EVPN Configuration Overview

In this scenario, multidestination traffic is handled using PIM. In order to use BGP EVPN as the control plane, you need nv overlay evpn global configuration.

MP-BGP is known for its scalability, multitenancy, and routing policy capabilities. BGP EVPN address family (address-family l2vpn evpn) carries the MAC, IP, Network, VRF, and VTEP information of the hosts. Once a VTEP leans about a host behind it, BGP EVPN distributes this information to all other BGP EVPN neighbours. We have already discussed MP-BGP in this post. But I am going to give a brief summary in our BGP EVPN context.

Route Distinguisher helps us differentiate between different routes stored in MP-BGP table. For separation and path efficiency, use a unique RD on a per-VRF, per- router basis.

Route Target – which happens to have a similar notation as RD – associates with a particular route which then allows that route placed in the VRF which imports that route.
  * Export route-target: what routes will go from VRF into BGP
  * Import route-target: what routes will go from BGP into VRF

NX-OS provides automated derivation of RDs and RTs for simplification. The snippet below is an example of the configuration in one of the spines followed by an example on one of the leafs.

```c
SPIN01(config)# nv overlay evpn ! Enables EVPN Control Plane in BGP
SPIN01(config)# feature bgp
SPIN01(config)# feature pim
SPIN01(config)# feature isis
SPIN01(config)# ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
SPIN01(config)# ip pim anycast-rp 192.168.0.254 192.168.0.1
SPIN01(config)# ip pim anycast-rp 192.168.0.254 192.168.0.2
SPIN01(config)# interface Ethernet1/1-4
SPIN01(config-if-range)# description FABRIC
SPIN01(config-if-range)# medium p2p
SPIN01(config-if-range)# ip unnumbered loopback0
SPIN01(config-if-range)# ip router isis UNDERLAY
SPIN01(config-if-range)# ip pim sparse-mode
SPIN01(config-if-range)# no shutdown
SPIN01(config-if-range)# exit
SPIN01(config)# interface loopback0
SPIN01(config-if)# ip address 192.168.0.1/32
SPIN01(config-if)# ip router isis UNDERLAY
SPIN01(config-if)# ip pim sparse-mode
SPIN01(config-if)# exit
SPIN01(config)# interface loopback254
SPIN01(config-if)# ip address 192.168.0.254/32
SPIN01(config-if)# ip router isis UNDERLAY
SPIN01(config-if)# ip pim sparse-mode
SPIN01(config-if)# exit
SPIN01(config)# router isis UNDERLAY
SPIN01(config-router)# net 49.0000.0000.0001.00
SPIN01(config-router)# is-type level-2
SPIN01(config-router)# exit
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
```c
LEAF01(config)# nv overlay evpn
LEAF01(config)# feature bgp
LEAF01(config)# feature pim
LEAF01(config)# feature isis
LEAF01(config)# feature vn-segment-vlan-based
LEAF01(config)# feature nv overlay ! Enables VTEP (Only required on leaf)
LEAF01(config)# ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
LEAF01(config)# vlan 100
LEAF01(config-vlan)# vn-segment 20100
LEAF01(config-vlan)# vlan 200
LEAF01(config-vlan)# vn-segment 20200
LEAF01(config-vlan)# exit
LEAF01(config)# interface nve1
LEAF01(config-if-nve)# no shutdown
LEAF01(config-if-nve)# host-reachability protocol bgp
LEAF01(config-if-nve)# source-interface loopback1
LEAF01(config-if-nve)# member vni 20100
LEAF01(config-if-nve-vni)# mcast-group 239.1.1.100
LEAF01(config-if-nve-vni)# member vni 20200
LEAF01(config-if-nve-vni)# mcast-group 239.1.1.200
LEAF01(config-if-nve-vni)# exit
LEAF01(config-if-nve)# interface Ethernet1/1-2
LEAF01(config-if-range)# description FABRIC
LEAF01(config-if-range)# no switchport
LEAF01(config-if-range)# medium p2p
LEAF01(config-if-range)# ip unnumbered loopback0
LEAF01(config-if-range)# ip router isis UNDERLAY
LEAF01(config-if-range)# ip pim sparse-mode
LEAF01(config-if-range)# no shutdown
LEAF01(config-if-range)# exit
LEAF01(config)# interface Ethernet1/41
LEAF01(config-if)# switchport access vlan 100
LEAF01(config-if)# spanning-tree port type edge
LEAF01(config-if)# interface loopback0
LEAF01(config-if)# ip address 192.168.1.1/32
LEAF01(config-if)# ip router isis UNDERLAY
LEAF01(config-if)# ip pim sparse-mode
LEAF01(config-if)# interface loopback1
LEAF01(config-if)# description NVE
LEAF01(config-if)# ip address 192.168.250.1/32
LEAF01(config-if)# ip router isis UNDERLAY
LEAF01(config-if)# ip pim sparse-mode
LEAF01(config-if)# exit
LEAF01(config)# router isis UNDERLAY
LEAF01(config-router)# net 49.0000.0000.1001.00
LEAF01(config-router)# is-type level-2
LEAF01(config-router)# router bgp 65000
LEAF01(config-router)# template peer-policy RR_PEER_POLICY
LEAF01(config-router-ptmp)# send-community extended
LEAF01(config-router-ptmp)# neighbor 192.168.0.1
LEAF01(config-router-neighbor)# remote-as 65000
LEAF01(config-router-neighbor)# update-source loopback0
LEAF01(config-router-neighbor)# address-family l2vpn evpn
LEAF01(config-router-neighbor-af)# send-community extended
LEAF01(config-router-neighbor-af)# neighbor 192.168.0.2
LEAF01(config-router-neighbor)# remote-as 65000
LEAF01(config-router-neighbor)# update-source loopback0
LEAF01(config-router-neighbor)# address-family l2vpn evpn
LEAF01(config-router-neighbor-af)# send-community extended
LEAF01(config)# evpn
LEAF01(config-evpn)# vni 20100 l2
LEAF01(config-evpn-evi)# rd auto
LEAF01(config-evpn-evi)# route-target import auto
LEAF01(config-evpn-evi)# route-target export auto
LEAF01(config-evpn-evi)# vni 20200 l2
LEAF01(config-evpn-evi)# rd auto
LEAF01(config-evpn-evi)# route-target import auto
LEAF01(config-evpn-evi)# route-target export auto
```
## Workshop

In this scenario, we will elaborate and modify our first lab in the previous post to work with VXLAN Flood and Learn as well as with multicast instead of Ingress Replication to handle BUM traffic. In this scenario, IS-IS is configured for underlay:

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232261114-774992f1-bed8-4042-b95a-5de440b84077.jpg" alt="VXLAN Fabric Multicast with MP-BGP Control Plane Workshop">
  <figcaption>VXLAN Fabric Multicast with MP-BGP Control Plane Workshop</figcaption>
</figure>


<details>
 
<summary>SPINE01</summary>

```elixir
nv overlay evpn
feature bgp
feature pim
feature isis

ip pim rp-address 192.168.0.254
ip pim anycast-rp 192.168.0.254 192.168.0.1
ip pim anycast-rp 192.168.0.254 192.168.0.2

interface Ethernet1/1
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/2
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/3
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/4
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface loopback0
  ip address 192.168.0.1/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback254
  ip address 192.168.0.254/32
  ip router isis UNDERLAY
  ip pim sparse-mode

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
feature pim
feature isis
ip pim rp-address 192.168.0.254
ip pim anycast-rp 192.168.0.254 192.168.0.1
ip pim anycast-rp 192.168.0.254 192.168.0.2

interface Ethernet1/1
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/2
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/3
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/4
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface loopback0
  description UNDERLAY
  ip address 192.168.0.2/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback254
  ip address 192.168.0.254/32
  ip router isis UNDERLAY
  ip pim sparse-mode
icam monitor scale

line console
line vty
router isis UNDERLAY
  net 49.0000.0000.0002.00
  is-type level-2
```
</details>

<details>

<summary>LEAF01</summary>
 
```elixir
nv overlay evpn
feature bgp
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.1/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.1/32
  ip router isis UNDERLAY
  ip pim sparse-mode

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
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.2/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.2/32
  ip router isis UNDERLAY
  ip pim sparse-mode

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
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/41
  switchport access vlan 200
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.3/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.3/32
  ip router isis UNDERLAY
  ip pim sparse-mode

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
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200

interface Ethernet1/1
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/2
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/41
  switchport access vlan 200
  spanning-tree port type edge

interface loopback0
  ip address 192.168.1.4/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.4/32
  ip router isis UNDERLAY
  ip pim sparse-mode

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