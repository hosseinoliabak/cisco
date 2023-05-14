# VXLAN Integrated Route and Bridge (IRB)
Integrated Route and Bridge (IRB) simply means first-hop routing. We have two different semantics for IRB:
  * Asymmetric IRB: We have one L2VNI for each subnet we want to route between.
  * Symmetric IRB: We have one L3VNI associated with the VRF which is used for all routed traffic

### Asymmetric IRB

For now, forget the VXLAN and imagine you connect two switches via an L2 trunk link containing VLAN A and B. Each switch has SVI on both VLANs, the gateway for endpoints behind the respective VLAN. Endpoint A connected to Switch A wants to communicate to Endpoint B connected to Switch B.

  * Switch A receives the traffic (Bridge operation) from Endpoint A
  * From VLAN A to VLAN B it then performs routing (Route Operation), forwards to Switch B
  * Switch B performs the MAC lookup and forwards the traffic out the port connected to Endpoint B (Bridge operation).

Bringing the aforementioned trunk example to VXLAN by replacing VLAN A and B over trunk with L2VNI A and B.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232263971-dd1393a5-f7df-4e33-b3eb-bb9f505a2ae5.svg" alt="Asymmetric IRB: bridge-route-bridge">
  <figcaption>Figure 1: Asymmetric IRB: bridge-route-bridge</figcaption>
</figure>

Asymmetric IRB can be done with or without BGP EVPN as the control plane.

### Symmetric IRB
Let’s elaborate on our legacy trunk example. Now, remove the trunk link between your switches in the previous example then by assuming the switches are MLS, replace that trunk link with an L3 interface. On each switch, you need to issue routing for VLAN A and B to point to the next switch. Here would be the operation:

  * Endpoint A sends the packet to Switch A (Bridge operation).
  * Switch A performs RIB lookup then forwards to Switch B over the L3 link (Route operation).
  * Switch B receives the traffic and performs the route lookup towards the connected destination (Route Operation).
  * Switch B forwards the packet to Endpoint B (Bridge Operation).

The example above made it simpler to understand the Bridge-Route-Route-Bridge operation between VTEPs with a dedicated L3VNI (which plays a role like our L3 interface in the example above).

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232263975-7bf783ba-354b-43de-98e1-16c04ae611d6.svg" alt="Symmetric IRB: bridge-route-route-bridge" /> <br />
  <figcaption>Figure 2: Symmetric IRB: bridge-route-route-bridge</figcaption>
</figure>

Since you need to configure L3VNI, you need to have BGP EVPN for symmetric routing.

```c
LEAF01(config)# nv overlay evpn
LEAF01(config)# feature bgp
LEAF01(config)# feature vn-segment-vlan-based
LEAF01(config)# feature nv overlay
LEAF01(config)# feature fabric forwarding
LEAF01(config)# feature interface-vlan
LEAF01(config)# vlan 3967
LEAF01(config-vlan)# vn-segment 23967
LEAF01(config-vlan)# name L3VNI
LEAF01(config-vlan)# vrf context SMENODE
! VNI 23967 is used for IVR.
! This L3VNI will be in VXLAN header as tenant identifier:
LEAF01(config-vrf)# vni 23967
! RD is in the form of <BGP RID>:<VRF ID>
LEAF01(config-vrf)# rd auto
LEAF01(config-vrf)# address-family ipv4 unicast
! RT is to export/import L2VPN EVPN NLRIs
! RT is in the form of <BGP ASN>:<VNI ID>
LEAF01(config-vrf-af-ipv4)# route-target both auto evpn
LEAF01(config-vrf-af-ipv4)# route-target both auto
! L3VNI needs SVI:
LEAF01(config-if)# interface Vlan3967
LEAF01(config-if)# no shutdown
LEAF01(config-if)# vrf member SMENODE
! Non-bridging (Enables Routing):
LEAF01(config-if)# ip forward
LEAF01(config-if)# exit
LEAF01(config)# interface nve1
LEAF01(config-if-nve)# no shutdown
LEAF01(config-if-nve)# host-reachability protocol bgp
LEAF01(config-if-nve)# source-interface loopback1
LEAF01(config-if-nve)# member vni 20100
LEAF01(config-if-nve-vni)# mcast-group 239.1.1.100
LEAF01(config-if-nve-vni)# member vni 20200
LEAF01(config-if-nve-vni)# mcast-group 239.1.1.200
! Then enable L3VNI and associate it to VRF:
LEAF01(config-if-nve-vni)# member vni 23967 associate-vrf
LEAF01(config)# router bgp 65000
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
```

### Distributed Anycast Gateway
For the endpoints mobility, we are not intending to change the gateway IP address of the endpoints when they roam from one rack to another. The same gateway for a subnet can coexist at multiple leafs. The distributed anycast gateway shares the same MAC address across all fabric. That is the Anycast Gateway Mac address (AGM) for all subnets. Each subnet share its own unique default gateway IP.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232264079-036d9432-7e9e-40a6-8ee4-5120aff638be.svg" alt="Distributed Anycast Gateway">
  <figcaption>Figure 3: Distributed Anycast Gateway</figcaption>
</figure>


With command `fabric forwarding anycast-gateway-mac 0001.0001.0001` (where 0001.0001.0001 is an example MAC address), you will configure the anycast Gateway MAC address globally. Then, you will require to configure the anycast SVI.

```c
LEAF01(config)# fabric forwarding anycast-gateway-mac 0001.0001.0001
LEAF01(config)# interface Vlan100
LEAF01(config-if)# no shutdown
LEAF01(config-if)# vrf member SMENODE
LEAF01(config-if)# ip address 192.168.100.254/24
LEAF01(config-if)# fabric forwarding mode anycast-gateway
```

## Workshop
In this workshop we will continue working on our previous workshop, but with introducing L3VNI Configuration:

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232261114-774992f1-bed8-4042-b95a-5de440b84077.jpg" alt="VXLAN MPBGP EVPN L3VNI Workshop">
  <figcaption>Figure 4: VXLAN MPBGP EVPN L3VNI Workshop</figcaption>
</figure>


#### Configuration

<details>
 
<summary>SPINE01</summary>

```elixir
hostname SPINE01

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
hostname SPINE02

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
  ip address 192.168.0.2/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback254
  ip address 192.168.0.254/32
  ip router isis UNDERLAY
  ip pim sparse-mode

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
hostname LEAF01
!
nv overlay evpn
feature bgp
feature fabric forwarding
feature interface-vlan
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

fabric forwarding anycast-gateway-mac 0001.0001.0001
!
vlan 100,200,3967
!
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  vn-segment 23967
  name L3VNI
!
vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto evpn 
    !generate route into L2VPN EVPN table
    route-target both auto 
    !route within IPv4 unicast BGP table
    !Traffic leaves the fabric
    !If ASR wantes to come into the fabric needs to know where the VLAN is
!
interface Vlan100
  no shutdown
  vrf member SMENODE
  ip address 192.168.100.254/24
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  ip address 192.168.200.254/24
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  ip forward
!
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200
  member vni 23967 associate-vrf

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
interface Ethernet1/42
  switchport access vlan 200
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
!
router bgp 65000
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
!
! The following is to advertise the L2VNI...
! ...within the BGP EVPN address family.
! Note that vrf is for L3VNI; evpn is for L2VNI
evpn
  vni 20100 l2
    rd auto
    route-target both auto
  vni 20200 l2
    rd auto
    route-target both auto
```

</details>

<details>

<summary>LEAF02</summary>

```elixir
hostname LEAF02
!
nv overlay evpn
feature bgp
feature fabric forwarding
feature interface-vlan
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

fabric forwarding anycast-gateway-mac 0001.0001.0001
!
vlan 100,200,3967
!
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  vn-segment 23967
  name L3VNI
!
vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto evpn
    !generate route into L2VPN EVPN table
    route-target both auto
    !route within IPv4 unicast BGP table
    !Traffic leaves the fabric  
    !If ASR wantes to come into the fabric needs to know where the VLAN is
!
interface Vlan100
  no shutdown
  vrf member SMENODE
  ip address 192.168.100.254/24
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  ip address 192.168.200.254/24
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  ip forward
!
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200
  member vni 23967 associate-vrf

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
interface Ethernet1/42
  switchport access vlan 200
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
!
router bgp 65000
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
!
! The following is to advertise the L2VNI...
! ...within the BGP EVPN address family.
! Note that vrf is for L3VNI; evpn is for L2VNI
evpn
  vni 20100 l2
    rd auto
    route-target both auto
  vni 20200 l2
    rd auto
    route-target both auto
```
</details>

<details>

<summary>LEAF03</summary>

```elixir
hostname LEAF03
!
nv overlay evpn
feature bgp
feature fabric forwarding
feature interface-vlan
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

fabric forwarding anycast-gateway-mac 0001.0001.0001
!
vlan 100,200,3967
!
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  vn-segment 23967
  name L3VNI
!
vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto evpn 
    !generate route into L2VPN EVPN table
    route-target both auto 
    !route within IPv4 unicast BGP table
    !Traffic leaves the fabric
    !If ASR wantes to come into the fabric needs to know where the VLAN is
!
interface Vlan100
  no shutdown
  vrf member SMENODE
  ip address 192.168.100.254/24
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  ip address 192.168.200.254/24
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  ip forward
!
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200
  member vni 23967 associate-vrf

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
interface Ethernet1/42
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
!
router bgp 65000
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
!
! The following is to advertise the L2VNI...
! ...within the BGP EVPN address family.
! Note that vrf is for L3VNI; evpn is for L2VNI
evpn
  vni 20100 l2
    rd auto
    route-target both auto
  vni 20200 l2
    rd auto
    route-target both auto
```
</details>

<details>

<summary>LEAF04</summary>

```elixir
hostname LEAF04
!
nv overlay evpn
feature bgp
feature fabric forwarding
feature interface-vlan
feature pim
feature isis
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254

fabric forwarding anycast-gateway-mac 0001.0001.0001
!
vlan 100,200,3967
!
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  vn-segment 23967
  name L3VNI
!
vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto evpn 
    !generate route into L2VPN EVPN table
    route-target both auto 
    !route within IPv4 unicast BGP table
    !Traffic leaves the fabric
    !If ASR wantes to come into the fabric needs to know where the VLAN is
!
interface Vlan100
  no shutdown
  vrf member SMENODE
  ip address 192.168.100.254/24
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  ip address 192.168.200.254/24
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  ip forward
!
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 20100
    mcast-group 239.1.1.100
  member vni 20200
    mcast-group 239.1.1.200
  member vni 23967 associate-vrf

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
interface Ethernet1/42
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
!
router bgp 65000
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
!
! The following is to advertise the L2VNI...
! ...within the BGP EVPN address family.
! Note that vrf is for L3VNI; evpn is for L2VNI
evpn
  vni 20100 l2
    rd auto
    route-target both auto
  vni 20200 l2
    rd auto
    route-target both auto
```
</details>
