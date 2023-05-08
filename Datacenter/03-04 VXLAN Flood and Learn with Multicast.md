# VXLAN Flood and Learn with Multicast

In the previous post, we talked about F&L with Ingress Replication (AKA head-end replication) to handle multi-destination traffic. In this section, we are going to discuss F&L with PIM to handle multi-destination traffic. So, your complete bipartite graph must support PIM. This is why in this post, we happened to choose loopback 254 on the spines for our PIM Anycast RP.

With multicast, VTEP V1 determines that the multidestination traffic needs to be sent to all members of VNI 20100. (As with IR, at the configuration time, when you configure the VNI, you also map it to a particular multicast group). VTEP V1 then encapsulates the packet with VXLAN header.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232262481-cec126c2-f684-498d-aabf-88b0b68771d6.svg" alt="VXLAN Flood and Learn with Multicast">
  <figcaption>Figure 1: VXLAN Flood and Learn with Multicast</figcaption>
</figure>


VTEP V1 then sends out the packet toward the IP core. The multicast tree forwards the packet until it reaches all interested receivers.

VTEP V3 receives and then de-encapsulates the VXLAN packet; because VTEP V3 has VNI 20100 and is part of subscribers for Multicast Group 239.1.1.2.

```c
LEAF01(config)# interface loopback1
LEAF01(config-if)# description VXLAN-VTEP-NVE-IF
LEAF01(config-if)# ip address 192.168.250.1/32
LEAF01(config-if)# ip router isis UNDERLAY
LEAF01(config-if)# ip pim sparse-mode
LEAF01(config)# feature nv overlay
LEAF01(config)# feature vn-segment-vlan-based
LEAF01(config)# vlan 100
LEAF01(config-vlan)# vn-segment 20100
LEAF01(config)# interface nve1
LEAF01(config-if-nve)# no shutdown
LEAF01(config-if-nve)# source-interface loopback1
LEAF01(config-if-nve)# member vni 20100
LEAF01(config-if-nve-vni)# mcast-group 239.1.1.100
```
## Workshop

In this scenario, we will elaborate and modify our first lab in the previous post to work with VXLAN Flood and Learn as well as with multicast instead of Ingress Replication to handle BUM traffic. In this scenario, IS-IS is configured for underlay:

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232261114-774992f1-bed8-4042-b95a-5de440b84077.jpg" alt="VXLAN F&L and Multicast Workshop">
  <figcaption>Figure 2: VXLAN Flood and Learn with Multicast Workshop</figcaption>
</figure>

#### Configuration

<details>
 
<summary>SPINE01</summary>

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
  ip address 192.168.0.1/32
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
  net 49.0000.0000.0001.00
  is-type level-2
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
feature pim
feature isis
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254
vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
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

interface loopback0
  description UNDERLAY
  ip address 192.168.1.1/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.1/32
  ip router isis UNDERLAY
  ip pim sparse-mode
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1001.00
  is-type level-2
```

</details>

<details>

<summary>LEAF02</summary>

```elixir
feature pim
feature isis
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254
vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
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

interface loopback0
  description UNDERLAY
  ip address 192.168.1.2/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.2/32
  ip router isis UNDERLAY
  ip pim sparse-mode
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1002.00
  is-type level-2
```
</details>

<details>

<summary>LEAF03</summary>

```elixir
feature pim
feature isis
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254
vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
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

interface loopback0
  description UNDERLAY
  ip address 192.168.1.3/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.3/32
  ip router isis UNDERLAY
  ip pim sparse-mode
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1003.00
  is-type level-2
```
</details>

<details>

<summary>LEAF04</summary>

```elixir
feature pim
feature isis
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address 192.168.0.254
vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
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

interface loopback0
  description UNDERLAY
  ip address 192.168.1.4/32
  ip router isis UNDERLAY
  ip pim sparse-mode

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.4/32
  ip router isis UNDERLAY
  ip pim sparse-mode
icam monitor scale

router isis UNDERLAY
  net 49.0000.0000.1004.00
  is-type level-2
  ```
</details>