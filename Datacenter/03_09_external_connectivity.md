# VXLAN Fabric External Connectivity
Users connect to our data centers above the application layer. They are typically external to data centre resources. In this post, we will discuss how to connect our VXLAN-based data centre to an external Layer 3 network as well as the external connectivity to a Layer 2 network.

  * Layer 3 could involve: the Internet, WAN, branch, other data centres, or the campus network
  * Layer 2 could involve: Integrate CE to VXLAN BGP EVPN, Layer 4-7 service insertion L2 connectivity, non-IP communication – for example, Reverse ARP (RARP) for endpoint mobility

### VXLAN External Connectivity Placement
With leaf and spine topology, the node which connects the VXLAN fabric to the external (Layer 2 or Layer 3) is called the border node. You can terminate the external connection to either spine or leaf switches. Placing the external connectivity on leaf switches is desirable. Because if you place the external connectivity on the spines, they become VTEPs.

#### Border Spine:

* Adding a switch to (scaling out) spine layer impacts external connectivity.
* You combine the transport for both north-south and east-west traffic.
* The spine is now a VTEP which performs VXLAN encapsulation and decapsulation for north-south flow.
* For east-west flows, the spine continues to provide underlay functionality.

#### Border Leaf:

* Border leaf is responsible for north-south traffic.
* Adding a leaf node does not impact north-south traffic.
* We discuss border leaf throughout this course.

### External Layer 3 Connectivity

Let’s review the options for physically cabling the border leaf to the external network.
* Full mesh: most common and recommended
* U-shaped: not recommended and we won’t discuss it here.

![vxlan-external-Full-mesh drawio-1](https://user-images.githubusercontent.com/31813625/232337073-648f071c-df59-4c40-af91-80df7140dbd4.svg)
<br /> *VXLAN Fabric Full-Mesh Border Connectivity*

The border leaf is still a VTEP. So configure it as VTEP. In the snippet below, I am going to show you the BGP configuration as far as concerned for external connectivity.

Here is a sample BGP configuration on Border Leaf:

```c
BLEAF01(config)# router bgp 65000
BLEAF01(config-router)# address-family l2vpn evpn
! iBGP neighbours (Spines)
BLEAF01(config-router-af)# neighbor 192.168.0.1
BLEAF01(config-router-neighbor)# remote-as 65000
BLEAF01(config-router-neighbor)# update-source loopback0
BLEAF01(config-router-neighbor)# address-family l2vpn evpn
BLEAF01(config-router-neighbor-af)# send-community extended
BLEAF01(config-router-neighbor-af)# neighbor 192.168.0.2
BLEAF01(config-router-neighbor)# remote-as 65000
BLEAF01(config-router-neighbor)# update-source loopback0
BLEAF01(config-router-neighbor)# address-family l2vpn evpn
BLEAF01(config-router-neighbor-af)# send-community extended
BLEAF01(config-router-neighbor-af)# vrf SMENODE
! The next two lines are only needed when you want to send an aggregate
! route instead of whole individual host routes
BLEAF01(config-router-vrf)# address-family ipv4 unicast
BLEAF01(config-router-vrf-af)# aggregate-address 192.168.0.0/16 summary-only
BLEAF01(config-router-vrf-af)# maximum-paths 4
! eBGP neighbours
BLEAF01(config-router-vrf-af)# neighbor 198.51.100.1
BLEAF01(config-router-vrf-neighbor)# remote-as 19851100
BLEAF01(config-router-vrf-neighbor)# address-family ipv4 unicast
BLEAF01(config-router-vrf-neighbor-af)# send-community
BLEAF01(config-router-vrf-neighbor-af)# send-community extended
BLEAF01(config-router-vrf-neighbor-af)# neighbor 203.0.113.1
BLEAF01(config-router-vrf-neighbor)# remote-as 2030113
BLEAF01(config-router-vrf-neighbor)# address-family ipv4 unicast
BLEAF01(config-router-vrf-neighbor-af)# send-community
BLEAF01(config-router-vrf-neighbor-af)# send-community extended
```
Here is a sample configuration on the ISP (or external router)
```c
INET01(config)# router bgp 19851100
INET01(config-router)# neighbor 198.51.100.2 remote-as 65000
INET01(config-router)# neighbor 198.51.100.10 remote-as 65000
INET01(config-router)# address-family ipv4
INET01(config-router-af)# network 8.8.8.8 mask 255.255.255.255
INET01(config-router-af)# neighbor 198.51.100.2 activate
INET01(config-router-af)# neighbor 198.51.100.10 activate
```


![vxlan-external-U-Shaped drawio](https://user-images.githubusercontent.com/31813625/232337233-fe3b9a48-4d4c-42aa-a8dc-3dec5a80d630.svg)

*VXLAN Fabric U-Shaped Border Connectivity*
On the border leaf, we have all VRF instances. We then send the associated Layer 3 route from the EVPN network into the individual VRF routing tables toward the edge router.

External routers don’t need your host routes, you can aggregate at MP-BGP to BGP/IGP redistribution point.

### External Layer 2 Connectivity
When creating external L2 connectivity between your CE and VXLAN fabric, make sure your design does not introduce a loop.

![vxlan-external-L2 drawio-1](https://user-images.githubusercontent.com/31813625/232337376-04d304a5-704c-43f1-afac-7064baea03ff.svg)

*Layer 2 Connectivity with VXLAN Fabric*

## Workshop
In this workshop, we are building a full mesh external connectivity. I am trying to keep it very simple and basic. Note that there is no multi-tenancy.

![external-connectivity-workshop](https://user-images.githubusercontent.com/31813625/232337435-1fed07e1-cb8f-4132-8f97-93a5ac587dd0.jpg)

*VXLAN Fabric L3 External Connectivity Workshop*

<details>
 
<summary>LEAF01</summary>

```elixir

hostname LEAF01

nv overlay evpn
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001
ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
vlan 1,100,200,3967
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  name L3VNI
  vn-segment 23967

vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

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
hostname LEAF02

nv overlay evpn
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001
ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
vlan 1,100,200,3967
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  name L3VNI
  vn-segment 23967

vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

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

interface mgmt0
  vrf member management

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
hostname LEAF03

nv overlay evpn
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001
ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
vlan 1,100,200,3967
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  name L3VNI
  vn-segment 23967

vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

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
hostname LEAF04

nv overlay evpn
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac 0001.0001.0001
ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
vlan 1,100,200,3967
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200
vlan 3967
  name L3VNI
  vn-segment 23967

vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

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

<summary>SPINE01</summary>

```elixir
hostname SPINE01

nv overlay evpn
feature bgp
feature pim
feature isis

ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
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

interface Ethernet1/5
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/6
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
  neighbor 192.168.4.252
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.4.253
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

ip pim rp-address 192.168.0.254 group-list 224.0.0.0/4
ip pim ssm range 232.0.0.0/8
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

interface Ethernet1/5
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  ip pim sparse-mode
  no shutdown

interface Ethernet1/6
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
  neighbor 192.168.4.252
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
  neighbor 192.168.4.253
    inherit peer-session RR_PEER_SESSION
    address-family l2vpn evpn
      inherit peer-policy RR_PEER_POLICY 1
```
</details>

<details>

<summary>BLEAF01</summary>

```elixir
hostname BLEAF01

nv overlay evpn
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

ip pim ssm range 232.0.0.0/8
vlan 1,3967
vlan 3967
  vn-segment 23967

vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

interface Vlan3967
  no shutdown
  vrf member SMENODE
  ip forward

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

interface Ethernet1/11
  no switchport
  vrf member SMENODE
  ip address 198.51.100.2/29
  no shutdown

interface Ethernet1/12
  no switchport
  vrf member SMENODE
  ip address 203.0.113.2/29
  no shutdown

interface loopback0
  ip address 192.168.4.252/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.253.252/32
  ip router isis UNDERLAY
  ip pim sparse-mode

router isis UNDERLAY
  net 49.0000.0000.2001.00
  is-type level-2
router bgp 65000
  address-family l2vpn evpn
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
  vrf SMENODE
    address-family ipv4 unicast
      aggregate-address 192.168.0.0/16 summary-only
    neighbor 198.51.100.1
      remote-as 19851100
      address-family ipv4 unicast
        send-community
        send-community extended
    neighbor 203.0.113.1
      remote-as 2030113
      address-family ipv4 unicast
        send-community
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

<summary>BLEAF02</summary>

```elixir
hostname BLEAF02

nv overlay evpn
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

ip pim ssm range 232.0.0.0/8
vlan 1,3967
vlan 3967
  vn-segment 23967

vrf context SMENODE
  vni 23967
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

interface Vlan3967
  no shutdown
  vrf member SMENODE
  ip forward

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

interface Ethernet1/11
  no switchport
  vrf member SMENODE
  ip address 198.51.100.10/29
  no shutdown

interface Ethernet1/12
  no switchport
  vrf member SMENODE
  ip address 203.0.113.10/29
  no shutdown

interface mgmt0
  vrf member management

interface loopback0
  ip address 192.168.4.253/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.253.253/32
  ip router isis UNDERLAY
  ip pim sparse-mode

router isis UNDERLAY
  net 49.0000.0000.2002.00
  is-type level-2
router bgp 65000
  address-family l2vpn evpn
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
  vrf SMENODE
    address-family ipv4 unicast
      aggregate-address 192.168.0.0/16 summary-only
    neighbor 198.51.100.9
      remote-as 19851100
      address-family ipv4 unicast
        send-community
        send-community extended
    neighbor 203.0.113.9
      remote-as 2030113
      address-family ipv4 unicast
        send-community
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

<summary>INET01</summary>

```elixir
hostname INET01

interface Loopback0
 ip address 8.8.8.8 255.255.255.255

interface Loopback1
 ip address 1.1.1.1 255.255.255.255

interface Ethernet0/0
 ip address 198.51.100.1 255.255.255.252

interface Ethernet0/1
 ip address 198.51.100.9 255.255.255.252

router bgp 19851100
 bgp log-neighbor-changes
 neighbor 198.51.100.2 remote-as 65000
 neighbor 198.51.100.10 remote-as 65000
 address-family ipv4
  network 1.1.1.1 mask 255.255.255.255
  network 8.8.8.8 mask 255.255.255.255
  neighbor 198.51.100.2 activate
  neighbor 198.51.100.10 activate
 exit-address-family
```
</details>

<details>

<summary>INET02</summary>

```elixir
hostname INET02

interface Loopback0
 ip address 4.2.2.4 255.255.255.255

interface Ethernet0/0
 ip address 203.0.113.1 255.255.255.248

interface Ethernet0/1
 ip address 203.0.113.9 255.255.255.248

router bgp 2030113
 bgp log-neighbor-changes
 neighbor 203.0.113.2 remote-as 65000
 neighbor 203.0.113.10 remote-as 65000
 address-family ipv4
  network 4.2.2.4 mask 255.255.255.255
  neighbor 203.0.113.2 activate
  neighbor 203.0.113.10 activate
 exit-address-family
```
</details>
