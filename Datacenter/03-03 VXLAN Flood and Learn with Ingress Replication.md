# VXLAN Flood and Learn with Ingress Replication

In this post, I am going to explain how the **location-identity** mapping database populates using Flood and Learn method. Consider the image below where VTEP01 takes the original ARP L2 frame from the HOST01 and then adds VXLAN header which includes L2VNI 20100 (think of it as an overlay L2 interface between VTEPs). VTEP01 will forward into the VXLAN core.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232261065-c59a1dbb-26d7-4e92-8766-6b72a9fd61a1.svg" alt="VXLAN Flood and Learn with Ingress Replication">
  <figcaption>VXLAN Flood and Learn with Ingress Replication</figcaption>
</figure>


In this scenario, multidestination traffic is handled using IR (as opposed to PIM). At the configuration time, when you configure the VNI at a VTEP, you also map it to a set of appropriate peer IPs.


```c
LEAF01(config)# interface loopback1
LEAF01(config-if)# description VXLAN-VTEP-NVE-IF
LEAF01(config-if)# ip address 192.168.250.1/32
LEAF01(config-if)# ip router isis UNDERLAY
LEAF01(config)# feature nv overlay
LEAF01(config)# feature vn-segment-vlan-based
LEAF01(config)# vlan 100
LEAF01(config-vlan)# vn-segment 20100
LEAF01(config)# interface nve 1
LEAF01(config-if-nve)# no shutdown
LEAF01(config-if-nve)# source-interface loopback 1
LEAF01(config-if-nve)# member vni 20100
LEAF01(config-if-nve-vni)# ingress-replication protocol static 
LEAF01(config-if-nve-vni-ingr-rep)# peer-ip 192.168.250.2
LEAF01(config-if-nve-vni-ingr-rep)# peer-ip 192.168.250.3
```

VXLAN core which is unaware of the inner header (identity), forwards the VXLAN frame based on only outer header (location). So, in our case, VXLAN core – which is typically our spines, are unaware of endpoint addresses (identities).

VTEP02 and VTEP03 receive the VXLAN frame and then perform deencapsulation. They both will learn MAC address of HOST01 and they map it to VTEP01.

There are two other notes here to mention: the subsequent messages are unicast if VTEP01 finds HOST02 location and identity. If there were a way which could distribute the same location-identity mapping across all edges, then the need for flooding could be eliminated.


## Workshop

In this scenario, we will configure our very first VXLAN topology with Flood and Learn as well as with Ingress Replication for BUM traffic. In this scenario, IS-IS is configured for underlay:

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/232261114-774992f1-bed8-4042-b95a-5de440b84077.jpg" alt="VXLAN F&L and IR Workshop">
  <figcaption>VXLAN Flood and Learn with Ingress Replication Workshop</figcaption>
</figure>



<details>
 
<summary>SPINE01</summary>

```elixir
hostname SPINE01
feature isis

interface Ethernet1/1-4
  mtu 9216
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface loopback0
  description UNDERLAY
  ip address 192.168.0.1/32

router isis UNDERLAY
  net 49.0000.0000.0001.00
  is-type level-2
```
</details>

<details>

<summary>SPINE02</summary>

```elixir
hostname SPINE02
feature isis

interface Ethernet1/1-4
  mtu 9216
  description FABRIC
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface loopback0
  description UNDERLAY
  ip address 192.168.0.2/32

router isis UNDERLAY
  net 49.0000.0000.0002.00
  is-type level-2
```

</details>

<details>

<summary>LEAF01</summary>
 

```elixir
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  source-interface loopback1
  member vni 20100
    ingress-replication protocol static
      peer-ip 192.168.250.2
      peer-ip 192.168.250.3
      peer-ip 192.168.250.4
  member vni 20200
    ingress-replication protocol static
      peer-ip 192.168.250.2
      peer-ip 192.168.250.3
      peer-ip 192.168.250.4

interface Ethernet1/1-2
  mtu 9216
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/41
  switchport access vlan 100

interface loopback0
  description UNDERLAY
  ip address 192.168.1.1/32

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.1/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0000.0000.1001.00
  is-type level-2
```

</details>

<details>

<summary>LEAF02</summary>

```elixir
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  source-interface loopback1
  member vni 20100
    ingress-replication protocol static
      peer-ip 192.168.250.1
      peer-ip 192.168.250.3
      peer-ip 192.168.250.4
  member vni 20200
    ingress-replication protocol static
      peer-ip 192.168.250.1
      peer-ip 192.168.250.3
      peer-ip 192.168.250.4

interface Ethernet1/1-2
  mtu 9216
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/41
  switchport access vlan 100

interface loopback0
  description UNDERLAY
  ip address 192.168.1.2/32

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.2/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0000.0000.1002.00
  is-type level-2
```

</details>

<details>

<summary>LEAF03</summary>

```elixir
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  source-interface loopback1
  member vni 20100
    ingress-replication protocol static
      peer-ip 192.168.250.1
      peer-ip 192.168.250.2
      peer-ip 192.168.250.4
  member vni 20200
    ingress-replication protocol static
      peer-ip 192.168.250.1
      peer-ip 192.168.250.2
      peer-ip 192.168.250.4

interface Ethernet1/1-2
  mtu 9216
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/41
  switchport access vlan 200

interface loopback0
  description UNDERLAY
  ip address 192.168.1.3/32

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.3/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0000.0000.1003.00
  is-type level-2
```
</details>

<details>

<summary>LEAF04</summary>

```elixir
feature isis
feature vn-segment-vlan-based
feature nv overlay

vlan 100,200
vlan 100
  vn-segment 20100
vlan 200
  vn-segment 20200

interface nve1
  no shutdown
  source-interface loopback1
  member vni 20100
    ingress-replication protocol static
      peer-ip 192.168.250.1
      peer-ip 192.168.250.2
      peer-ip 192.168.250.3
  member vni 20200
    ingress-replication protocol static
      peer-ip 192.168.250.1
      peer-ip 192.168.250.2
      peer-ip 192.168.250.3

interface Ethernet1/1-2
  mtu 9216
  description FABRIC
  no switchport
  medium p2p
  ip unnumbered loopback0
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/42
  switchport access vlan 200

interface loopback0
  description UNDERLAY
  ip address 192.168.1.4/32

interface loopback1
  description VXLAN-VTEP-NVE-IF
  ip address 192.168.250.4/32
  ip router isis UNDERLAY

router isis UNDERLAY
  net 49.0000.0000.1004.00
  is-type level-2
```
</details>