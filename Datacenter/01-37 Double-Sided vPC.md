# Double-Sided vPC

Double-side vPC or Dual-sided vPC as known as back-to-back vPC or multilayer vPC is combination of multiple vPC layers.

In double-sided vPC we have two layers of vPC pairs. Traditionally, our access layer switches are in vPC whose southbound links connect to servers. Their northbound links connect to aggregation layer who are also in vPC. To ensure that BPDUs and LAG IDs are unique, each vPC has to have different domain-id. You can configure the vPCs with same domain IP, but you must manually configure different system IDs for each vPC pair.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235373839-6e4a131a-5678-4e88-ad7b-13e1991ae321.png" alt="Double-Sided vPC">
  <figcaption>Figure 1: Double-Sided vPC</figcaption>
</figure>

There is another use case for back-to-back vPC which is for Datacenters Interconnectivity. You can extend one Datacenter to another Datacenter without having full mesh connectivity between each layer. For Datacenter Interconnectivity, Cisco recommends that we have two MST regions one for each datacenter.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235373850-b08a57b6-b90d-4e13-ad4a-c48c72cbfe12.png" alt="vPC DCI Topology">
  <figcaption>Figure 2: vPC DCI Topology</figcaption>
</figure>

## Double-sided vPC Workshop

In this workshop, we are going to demonstrate back-to-back vPC configuration in three different domains (Figure 1). With domain 1 being aggregation layer, domain 2 and 3 being access layer.

#### Configuration

<details>
 
<summary>AGGR01</summary>

```elixir
feature lacp
feature vpc
interface mgmt 0
  ip address 192.168.1.1/24
exit
vpc domain 1
  peer-keepalive destination 192.168.1.2 source 192.168.1.1 vrf management
exit
interface ethernet 1/1-2
switchport
  switchport mode trunk
channel-group 1 mode active
  no shutdown
interface port-channel 1
switchport
  switchport mode trunk
  no shutdown
  vpc peer-link
interface ethernet 1/3-4
 switchport
 switchport mode trunk
 channel-group 11 mode active
 no shutdown
interface port-channel 11
 switchport
 switchport mode trunk
 vpc 11
interface ethernet 1/5-6
 switchport
 switchport mode trunk
 channel-group 12 mode active
 no shutdown
interface port-channel 12
 switchport
 switchport mode trunk
 vpc 12
```
</details>

<details>

<summary>AGGR02</summary>

```elixir
feature lacp
feature vpc
interface mgmt 0
  ip address 192.168.1.2/24
exit
vpc domain 1
 peer-keepalive destination 192.168.1.1 source 192.168.1.2 vrf management
exit
interface ethernet 1/1-2
switchport
  switchport mode trunk
channel-group 1 mode active
  no shutdown
interface port-channel 1
switchport
  switchport mode trunk
  no shutdown
  vpc peer-link
interface ethernet 1/3-4
 switchport
 switchport mode trunk
 channel-group 11 mode active
 no shutdown
interface port-channel 11
 switchport
 switchport mode trunk
 vpc 11
interface ethernet 1/5-6
 switchport
 switchport mode trunk
 channel-group 12 mode active
 no shutdown
interface port-channel 12
 switchport
 switchport mode trunk
 vpc 12
```
</details>

<details>

<summary>ACC01</summary>

```elixir
feature lacp
feature vpc
interface mgmt 0
  ip address 192.168.2.1/24
exit
vpc domain 2
  peer-keepalive destination 192.168.2.2 source 192.168.2.1 vrf management
exit
interface ethernet 1/1-2
switchport
  switchport mode trunk
channel-group 2 mode active
  no shutdown
interface port-channel 2
switchport
  switchport mode trunk
  no shutdown
  vpc peer-link
interface ethernet 1/3-4
switchport
  switchport mode trunk
channel-group 11 mode active
  no shutdown
interface port-channel 11
switchport
  switchport mode trunk
  no shutdown
  vpc 11
```
</details>

<details>

<summary>ACC02</summary>

```elixir
feature lacp
feature vpc
interface mgmt 0
  ip address 192.168.2.2/24
exit
vpc domain 2
  peer-keepalive destination 192.168.2.1 source 192.168.2.2 vrf management
exit
interface ethernet 1/1-2
switchport
  switchport mode trunk
channel-group 2 mode active
  no shutdown
interface port-channel 2
switchport
  switchport mode trunk
  no shutdown
  vpc peer-link
interface ethernet 1/3-4
switchport
  switchport mode trunk
channel-group 11 mode active
  no shutdown
interface port-channel 11
switchport
  switchport mode trunk
  no shutdown
  vpc 11
```
</details>

<details>

<summary>ACC03</summary>

```elixir
feature lacp
feature vpc
interface mgmt 0
  ip address 192.168.2.3/24
exit
vpc domain 3
  peer-keepalive destination 192.168.2.4 source 192.168.2.3 vrf management
exit
interface ethernet 1/1-2
switchport
  switchport mode trunk
channel-group 3 mode active
  no shutdown
interface port-channel 3
switchport
  switchport mode trunk
  no shutdown
  vpc peer-link
interface ethernet 1/5-6
switchport
  switchport mode trunk
channel-group 12 mode active
  no shutdown
interface port-channel 12
switchport
  switchport mode trunk
  no shutdown
  vpc 12
```
</details>

<details>

<summary>ACC04</summary>

```elixir
feature lacp
feature vpc
interface mgmt 0
  ip address 192.168.2.4/24
exit
vpc domain 3
  peer-keepalive destination 192.168.2.3 source 192.168.2.4 vrf management
exit
interface ethernet 1/1-2
switchport
  switchport mode trunk
channel-group 3 mode active
  no shutdown
interface port-channel 3
switchport
  switchport mode trunk
  no shutdown
  vpc peer-link
interface ethernet 1/5-6
switchport
  switchport mode trunk
channel-group 12 mode active
  no shutdown
interface port-channel 12
switchport
  switchport mode trunk
  no shutdown
  vpc 12
```
</details>