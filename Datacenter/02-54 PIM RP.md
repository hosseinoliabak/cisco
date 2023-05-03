# PIM Rendezvous Point

With shared tree, multicast distribution roots at RP.

### RP Configuration

You can configure – for different groups – as many rendezvous points as you like. But you must note that all routers in the multicast domain agree on the RP for the group. There are different ways that RPs are defined:
  * Static RP: You configure the address of RP on every single router in the multicast domain.
  * Dynamic Active/Standby RP(s):
    * Bootstrap Router (BSR): Makes sure that all routers in the multicast domain have the same RP as the BSR.
    * Auto-RP: Cisco proprietary which was used prior to BSR standard.
  * Active/Active RPs
    * Anycast RP
    * Phantom RP

> Note: Static RP can coexist with Dynamic RP. But dynamic RP takes precedence over the static RP.
> Note: You cannot configure BSR and Auto-RP at the same time.

### BSR
Some routers are configured as RPs (candidate RPs) and advertise themselves as such to other PIM routers. The BSR receives this information and distributes the RP-to-group mapping to downstream routers who are listening to these advertisements (224.0.0.13).

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235809486-64266e6a-10a4-44c2-9fb0-d9174f424211.png" alt="Bootstrap Router (BSR)">
  <figcaption>Figure 1: Bootstrap Router (BSR)</figcaption>
</figure>
<p>&nbsp</p>   

```c
! On RP Candidate
N9K01(config)# ip pim rp-candidate l1 group-list 224.0.0.0/4

! On BSR
N9K03(config)# ip pim bsr bsr-candidate Ethernet1/1

! On Downstream routers
N9K04(config)# ip pim bsr listen
```

### Auto RP

Auto-RP propagates RP information via Auto-RP messages. RPs (Candidate RPs) announce their availability to the mapping agents who is listening on 224.0.1.39. Downstream routers listen to 224.0.1.40 for mapping advertisements from mapping agent.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235810359-dcd19063-c773-4182-a925-ff12aed6f6d3.png" alt="Auto-RP">
  <figcaption>Figure 2: Auto-RP</figcaption>
</figure>
<p>&nbsp</p>   


## RP Redundancy

There are two RP redundancy mechanisms used with static RPs which I want to cover here:
  * Anycast RP
  * Phantom RP

### Anycast RP

Anycast RP achieves active/active RP distribution for PIM-SM. Anycast requires both RPs to use the same IP address. You assign a duplicate Loopback address and advertise into IGP domain. PIM Register and Join messages go to the closest unicast metric RP in the topology. What if PIM Register is sent to one anycast RP, and PIM Join is sent to another? There are two variants for Anycast RP:

  * Multicast Source Discovery Protocol (MSDP) (RFC 3446)
  * PIM Anycast RP (RFC 4610)

#### Multicast Source Discovery Protocol (MSDP)

MSDP (RFC 3618) was used for Inter-domain source active communication in the Internet. We can make use of this intra domain. MSDP listens to active sources in a network (PIM registers) and shares those sources with only explicitly configured neighbors (on TCP port 639). RPs then join to those active sources. We run MSDP between the RPs. Each device has a globally routable Loopback plus the anycast Loopback.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235809829-c772e497-7372-4b1f-971b-fe15da93911d.png" alt="Anycast RP with MSDP">
  <figcaption>Figure 3: Anycast RP with MSDP</figcaption>
</figure>
<p>&nbsp</p>   

```elixir
! On RP1
! Loopback0 is not anycast loopback; not pim enabled
N9K01(config)# feature msdp
N9K01(config)# ip msdp originator-id loopback0
N9K01(config)# ip msdp peer 192.168.0.2 connect-source loopback0
N9K01(config)# ip pim rp-address 192.168.0.254
! 192.168.0.254 is IP address of Loopback 1 which is PIM SM enabled

! On RP2
! Loopback0 is not anycast loopback; not pim enabled
N9K01(config)# feature msdp
N9K02(config)# ip msdp originator-id loopback 0
N9K02(config)# ip msdp peer 192.168.0.1 connect-source loopback 0
N9K01(config)# ip pim rp-address 192.168.0.254
! 192.168.0.254 is IP address of Loopback 1 which is PIM SM enabled
```

Verification:

```elixir
N9K01# show ip msdp summary
MSDP Peer Status Summary for VRF "default"
Local ASN: 0, originator-id: 192.168.0.1

Number of configured peers:  1
Number of established peers: 1
Number of shutdown peers:    0

Peer            Peer        Connection      Uptime/   Last msg  (S,G)s
Address         ASN         State           Downtime  Received  Received
192.168.0.2     0           Established     00:11:30  00:00:24  1

N9K02# show ip msdp summary
MSDP Peer Status Summary for VRF "default"
Local ASN: 0, originator-id: 192.168.0.2

Number of configured peers:  1
Number of established peers: 1
Number of shutdown peers:    0

Peer            Peer        Connection      Uptime/   Last msg  (S,G)s
Address         ASN         State           Downtime  Received  Received
192.168.0.1     0           Established     00:11:31  00:00:47  1

N9K02# show ip msdp sa-cache
MSDP SA Route Cache for VRF "default" - 1 entries
Source       Group       RP            ASN      Uptime    In_MRIB   Peer address
172.16.1.2   239.0.1.2   192.168.0.1   0        00:02:19  True      192.168.0.1

N9K02# show ip mroute
IP Multicast Routing Table for VRF "default"

(*, 232.0.0.0/8), uptime: 00:13:38, pim ip
  Incoming interface: Null, RPF nbr: 0.0.0.0
  Outgoing interface list: (count: 0)

(*, 239.0.1.2/32), uptime: 00:13:19, pim ip
  Incoming interface: loopback1, RPF nbr: 192.168.0.254
  Outgoing interface list: (count: 1)
    Ethernet1/2, uptime: 00:13:16, pim

(172.16.1.2/32, 239.0.1.2/32), uptime: 00:13:27, ip pim msdp mrib
  Incoming interface: Ethernet1/1, RPF nbr: 10.1.1.9
  Outgoing interface list: (count: 1)
    Ethernet1/2, uptime: 00:13:16, pim
```

On downstream routers, you tell who the RP is, remember to give them the anycast loopback IP (not the Lo0 with the example above).

#### PIM Anycast RP

Let’s not enable MSDP feature. Let’s do anycast RP technic with using only PIM protocol. Let’s use PIM register message for synchronization. PIM Anycast RP uses PIM to build active/active distribution between the RPs. You need to configure a set of RP routers on each RP. Once one RP (let’s say RP1), receives Register message, this router adds (S,G) to its forwarding table, then it sends Register message to all other RPs listed in the set of RPs.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235810011-355c6377-f17f-4293-8aaf-3c0357fb7176.png" alt="Anycast RP with PIM Register Message">
  <figcaption>Figure 4: Anycast RP with PIM Register Message</figcaption>
</figure>
<p>&nbsp</p>   

Configuration on RPs

```elixir
N9K01(config)# ip pim anycast-rp 192.168.0.254 192.168.0.1
N9K01(config)# ip pim anycast-rp 192.168.0.254 192.168.0.2
N9K01(config)# ip pim rp-address 192.168.0.254
```

Configuration on non-RPs remain unchanged

```elixir
N9K03(config)# ip pim rp-address 192.168.0.254
```

Verification on an RP and a non-RP

```elixir
N9K01# show ip pim rp
PIM RP Status Information for VRF "default"
BSR disabled
Auto-RP disabled
BSR RP Candidate policy: None
BSR RP policy: None
Auto-RP Announce policy: None
Auto-RP Discovery policy: None

Anycast-RP 192.168.0.254 members:
  192.168.0.1*  192.168.0.2

RP: 192.168.0.254*, (0),
 uptime: 1d23h   priority: 255,
 RP-source: (local),
 group ranges:
 224.0.0.0/4
!
N9K03# show ip pim rp
rp        rp-hash
N9K03# sho ip pim rp
PIM RP Status Information for VRF "default"
BSR disabled
Auto-RP disabled
BSR RP Candidate policy: None
BSR RP policy: None
Auto-RP Announce policy: None
Auto-RP Discovery policy: None

RP: 192.168.0.254, (0),
 uptime: 1d23h   priority: 255,
 RP-source: (local),
 group ranges:
 224.0.0.0/4
```
Below, you can see the mroute is synchronized between the two RPs:

```elixir
N9K01# show ip mroute
IP Multicast Routing Table for VRF "default"

(*, 232.0.0.0/8), uptime: 00:07:53, pim ip
  Incoming interface: Null, RPF nbr: 0.0.0.0
  Outgoing interface list: (count: 0)

(*, 239.0.1.2/32), uptime: 00:07:29, pim ip
  Incoming interface: loopback1, RPF nbr: 192.168.0.254
  Outgoing interface list: (count: 1)
    Ethernet1/2, uptime: 00:07:29, pim

(172.16.1.2/32, 239.0.1.2/32), uptime: 00:00:04, pim mrib ip
  Incoming interface: Ethernet1/1, RPF nbr: 10.1.1.1, internal
  Outgoing interface list: (count: 1)
    Ethernet1/2, uptime: 00:00:04, pim
!
N9K02# show ip mroute
IP Multicast Routing Table for VRF "default"

(*, 232.0.0.0/8), uptime: 1d23h, pim ip
  Incoming interface: Null, RPF nbr: 0.0.0.0
  Outgoing interface list: (count: 0)

(*, 239.0.1.2/32), uptime: 00:08:25, pim ip
  Incoming interface: loopback1, RPF nbr: 192.168.0.254
  Outgoing interface list: (count: 0)

(172.16.1.2/32, 239.0.1.2/32), uptime: 00:00:08, pim mrib ip
  Incoming interface: Ethernet1/1, RPF nbr: 10.1.1.9, internal
  Outgoing interface list: (count: 0)
```

The Nexus platform supports both Anycast PIM and MSDP modes. So, This is a good choice to use Anycast PIM as we are not enabling MSDP feature.

### Phantom RP

PIM BiDir uses phantom RP or virtual RP for HA. It doesn’t need to be a physical router because there is no PIM Register message with PIM BiDir; so, there is no unicast packet to the RP address. Unlike PIM ASM, with BiDir the RP does not handle any control plane load and RP information. It is just an indication towards the root of the RPT.
