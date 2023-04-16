# vPC in VXLAN BGP EVPN
Your endpoints still connect through classic ethernet to the leafs. In most practical implementations, we need redundancy between the endpoints and the leaf switches. So, we typically leverage vPC to provide us MC-LAG. vPC allows an endpoint connects to two leafs. The endpoint sees those switches as a single switch to which connected via a single port channel link.

### vPC design consideration

1. Configure vPC. In this post, we previously talked about how we configure vPC.
2. Configure each leaf as a single VTEP. As usual, each VTEP has a dedicated NVE interface (PIP).
  * PIP is short for “Primary IP address.”
3. Configure anycast VTEP:
  * Configure a common virtual IP for VTEPs – This address is the next hop advertised in VXLAN.
    * You can achieve this by assigning a secondary IP address on your NVE loopback interface
  * So far, every vPC member should have a unique PIP and a shared VIP
    * PIP is used for Type 5
    * VIP is used for Type 2
4. Issue command advertise-pip under BGP L2VPN AFI and advertise virtual-rmac command under NVE interface.
  * Advertises the external prefixes with PIP of the individual VTEP as the next hop in route type 5.
    * Allows return traffic to reach that VTEP
  * Route type 2 (MAC/IP) continues using VIP.
  * PIP uses the switch Router MAC (non-transitive extended community).
  * VIP uses locally derived MAC based on VIP itself.

![VXLAN-VPC](https://user-images.githubusercontent.com/31813625/232338537-1982c012-12d4-475f-8a84-1f399cf77366.svg)
<br /> *vPC advertise-pip*

## Workshop
In this post, we are going to configure the leaf pairs as vPC.
* vPC domain 11 includes leaf-1 and leaf-2
* vPC domain 12 includes leaf-3 and leaf-4

![vxlan vpc workshop](https://user-images.githubusercontent.com/31813625/232338536-b47e59a8-c115-4119-b1fa-2991b311eb55.jpg)
<br /> *Workshop – vPC in VXLAN BGP EVPN*

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
feature lacp
feature vpc
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
vrf context management
vpc domain 11
  peer-switch
  peer-keepalive destination 192.168.10.4 source 192.168.10.3
  peer-gateway

interface Vlan100
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.100.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.200.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel11
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
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

interface Ethernet1/5
  switchport mode trunk
  spanning-tree port type network
  channel-group 11 mode active

interface Ethernet1/6
  switchport mode trunk
  spanning-tree port type network
  channel-group 11 mode active

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface Ethernet1/42
  switchport access vlan 200
  spanning-tree port type edge

interface mgmt0
  vrf member management
  ip address 192.168.10.3/24

interface loopback0
  ip address 192.168.1.1/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.1/32
  ip address 192.168.254.1/32 secondary
  ip router isis UNDERLAY
  ip pim sparse-mode

router isis UNDERLAY
  net 49.0000.0000.1001.00
  is-type level-2
router bgp 65000
  address-family l2vpn evpn
    advertise-pip
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

cfs eth distribute
nv overlay evpn
feature bgp
feature pim
feature isis
feature fabric forwarding
feature interface-vlan
feature vn-segment-vlan-based
feature lacp
feature vpc
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
vrf context management
vpc domain 11
  peer-switch
  peer-keepalive destination 192.168.10.3 source 192.168.10.4
  peer-gateway

interface Vlan1
  no ip redirects
  no ipv6 redirects

interface Vlan100
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.100.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.200.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel11
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
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

interface Ethernet1/5
  switchport mode trunk
  spanning-tree port type network
  channel-group 11 mode active

interface Ethernet1/6
  switchport mode trunk
  spanning-tree port type network
  channel-group 11 mode active

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface Ethernet1/42
  switchport access vlan 200
  spanning-tree port type edge

interface mgmt0
  vrf member management
  ip address 192.168.10.4/24

interface loopback0
  ip address 192.168.1.2/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.2/32
  ip address 192.168.254.1/32 secondary
  ip router isis UNDERLAY
  ip pim sparse-mode

router isis UNDERLAY
  net 49.0000.0000.1002.00
  is-type level-2
router bgp 65000
  address-family l2vpn evpn
    advertise-pip
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
feature lacp
feature vpc
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
vrf context management
vpc domain 12
  peer-switch
  peer-keepalive destination 192.168.10.6 source 192.168.10.5
  peer-gateway

interface Vlan100
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.100.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.200.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel12
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
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

interface Ethernet1/5
  switchport mode trunk
  spanning-tree port type network
  channel-group 12 mode active

interface Ethernet1/6
  switchport mode trunk
  spanning-tree port type network
  channel-group 12 mode active

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface Ethernet1/42
  switchport access vlan 200
  spanning-tree port type edge

interface mgmt0
  vrf member management
  ip address 192.168.10.5/24

interface loopback0
  ip address 192.168.1.3/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.3/32
  ip address 192.168.254.2/32 secondary
  ip router isis UNDERLAY
  ip pim sparse-mode
icam monitor scale

line console
line vty
router isis UNDERLAY
  net 49.0000.0000.1003.00
  is-type level-2
router bgp 65000
  address-family l2vpn evpn
    advertise-pip
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
feature lacp
feature vpc
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
vpc domain 12
  peer-switch
  peer-keepalive destination 192.168.10.5 source 192.168.10.6
  peer-gateway

interface Vlan100
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.100.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan200
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip address 192.168.200.254/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan3967
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel12
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
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

interface Ethernet1/5
  switchport mode trunk
  spanning-tree port type network
  channel-group 12 mode active

interface Ethernet1/6
  switchport mode trunk
  spanning-tree port type network
  channel-group 12 mode active

interface Ethernet1/41
  switchport access vlan 100
  spanning-tree port type edge

interface Ethernet1/42
  switchport access vlan 200
  spanning-tree port type edge

interface mgmt0
  vrf member management
  ip address 192.168.10.6/24

interface loopback0
  ip address 192.168.1.4/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description NVE
  ip address 192.168.250.4/32
  ip address 192.168.254.2/32 secondary
  ip router isis UNDERLAY
  ip pim sparse-mode

router isis UNDERLAY
  net 49.0000.0000.1004.00
  is-type level-2
router bgp 65000
  address-family l2vpn evpn
    advertise-pip
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
The following configurations remain intact compared to the last post.

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

interface mgmt0
  vrf member management
  ip address 192.168.10.1/24

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

interface mgmt0
  vrf member management
  ip address 192.168.10.2/24

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

interface mgmt0
  vrf member management
  ip address 192.168.10.7/24

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
feature lacp
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
vrf context management

interface Vlan1
  no ip redirects
  no ipv6 redirects

interface Vlan3967
  no shutdown
  vrf member SMENODE
  no ip redirects
  ip forward
  no ipv6 redirects

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
  ip address 192.168.10.8/24

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