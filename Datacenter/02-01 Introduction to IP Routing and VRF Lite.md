# Introduction to IP Routing and VRF Lite

 In this article, we'll provide an introduction to IP routing and VRF Lite, explaining what they are, how they work, and why they're important for building scalable and secure networks. Whether you're new to networking or looking to expand your knowledge, this post will give you a solid foundation for understanding these fundamental concepts.

## Network Layer Jobs

### IP Routing

One of the main roles of the Network layer is to forward packets between hosts. A host can send a packet to:

* **Itself:** Host can use it’s own interface IP or its loopback address to send a packet to itself.
* **Same local network:** When a host wants to send a packet to someone else, it checks if the destination is in the different network or in the same network (how? with its IP/Mask parameters). If the destination is in the same network, host sends an ARP request to find the destination MAC.
* **Different network:** If the destination is in the different network than our host, the host will send an ARP request to find the destination MAC address of it’s default gateway.

Another role of the network layer is to provide a/some loop-free path(s). There is three general classes of solutions which widely deployed to provide a loop-free path for packet forwarding:

Distance-Vector Protocols: calculates a loop-free paths hop by hop based on the path metric.
  * RIP/EIGRP
  * Interarea OSPF

Link-State Protocols: calculates a loop-free path based on a database synced between the routers in an area.
  * OSPF
  * IS-IS

Path-Vector Protocols: calculates a loop-free paths hop by hop based a record of previous hops
  * BGP

#### Routing Protocols vs Routed/Routable Protocols:

* Routing protocols: Static, OSPF, BGP, EIGRP
* Routed protocols or routable protocols: IP, IPv6

How does a router do classless routing?

1. It checks its routing table to find the longest prefix. If there is a tie, go to 2.
2. Least Administrative Distance (AD): If there is a tie, go to 3. Here are some ADs with Cisco software
  * Connected: 0
  * Static: 1
  * eBGP: 20
  * OSPF: 110
  * IS-IS: 115
  * iBGP 200
3. Metric: If there is a tie: then load-share the traffic.

### IP Routing: Static Route and Default route
You can configure a static route by specifying next-hop value and local outgoing interface (although both are not required together).

When the destination is 0.0.0.0/0, the static route is called static default route or simply a default route.

```go
N9K01# configure terminal
N9K01(config)# ip route 172.30.30.0/24 ethernet 1/1 100.64.1.1
```

### IP Routing: Virtual Routing and Forwarding (VRF) Lite

VRFs is to a router as VLAN is to a switch. VRF creates a new instance of routing table inside NX-OS. You need to create a VRF, then assign the interfaces to that VRF. The interfaces which are not in any VRF, are part of default VRF. Mgmt interface is by default part of Management VRF.

Remember, since we have different routing table, addressing as a result can overlap within different VRFs.

**Configuration:**

```go
! Create a VRF Context
N9K01(config)# vrf context CX-1
N9K01(config-vrf)# exit
! Assign L3 Interface to the VRF
N9K01(config)# interface loopback 101
N9K01(config-if)# no switchport
N9K01(config-if)# vrf member CX-1
! Create a static route into the VRF
N9K01(config-if)# vrf context CX-1
N9K01(config-vrf)# ip route 172.30.30.0/24 100.64.1.1
```

## VRF Workshop 2

In this workshop are are going to interconnect two nexus switches each with two VRFs

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235474911-e6de81b9-3bb6-45e2-b158-f6dcb12ee7f1.png" alt="VRF Lite Workshop">
  <figcaption>Figure 1: VRF Lite Workshop</figcaption>
</figure>

#### Configuration

<details>
 
<summary>N9K01 (On the left)</summary>

```go
vrf context CX-1
exit
vrf context CX-2
exit
interface loopback 101
  vrf member CX-1
  ip address 172.20.20.1/24
  no shutdown
interface loopback 102
  vrf member CX-2
  ip address 172.20.20.1/24
exit
interface ethernet 1/1
  vrf member CX-1
  ip address 100.64.1.0/31
  no shutdown
interface ethernet 1/2
no switchport
  vrf member CX-2
  ip address 100.64.2.0/31
  no shutdown
exit
vrf context CX-1
  ip route 172.30.30.0/24 100.64.1.1
vrf context CX-2
  ip route 172.31.31.0/24 100.64.2.1
```
</details>

<details>

<summary>N9K02 (On the right)</summary>

```go
vrf context CX-1
exit
vrf context CX-2
exit
interface loopback 101
  vrf member CX-1
  ip address 172.30.30.1/24
  no shutdown
interface loopback 102
  vrf member CX-2
  ip address 172.31.31.1/24
exit
interface ethernet 1/1
  vrf member CX-1
  ip address 100.64.1.1/31
  no shutdown
interface ethernet 1/2
no switchport
  vrf member CX-2
  ip address 100.64.2.1/31
  no shutdown
exit
vrf context CX-1
  ip route 172.20.20.0/24 100.64.1.0
vrf context CX-2
  ip route 172.20.20.0/24 100.64.2.0
```
</details>

#### Verification

Verify the routing table for each VRF:
<details>

<summary>N9K01 (On the left)</summary>

```go
N9K01# show ip route vrf CX-1
IP Route Table for VRF "CX-1"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

100.64.1.0/31, ubest/mbest: 1/0, attached
    *via 100.64.1.0, Eth1/1, [0/0], 00:12:42, direct
100.64.1.0/32, ubest/mbest: 1/0, attached
    *via 100.64.1.0, Eth1/1, [0/0], 00:12:42, local
172.20.20.0/24, ubest/mbest: 1/0, attached
    *via 172.20.20.1, Lo101, [0/0], 00:14:16, direct
172.20.20.1/32, ubest/mbest: 1/0, attached
    *via 172.20.20.1, Lo101, [0/0], 00:14:16, local
172.30.30.0/24, ubest/mbest: 1/0
    *via 100.64.1.1, [1/0], 00:06:02, static

N9K01# show ip route vrf CX-2
IP Route Table for VRF "CX-2"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

100.64.2.0/31, ubest/mbest: 1/0, attached
    *via 100.64.2.0, Eth1/2, [0/0], 00:11:36, direct
100.64.2.0/32, ubest/mbest: 1/0, attached
    *via 100.64.2.0, Eth1/2, [0/0], 00:11:36, local
172.20.20.0/24, ubest/mbest: 1/0, attached
    *via 172.20.20.1, Lo102, [0/0], 00:13:54, direct
172.20.20.1/32, ubest/mbest: 1/0, attached
    *via 172.20.20.1, Lo102, [0/0], 00:13:54, local
172.31.31.0/24, ubest/mbest: 1/0
    *via 100.64.2.1, [1/0], 00:06:08, static
```
</details>

Verify the end host reachability:

<details>

<summary>N9K02 (On the right)</summary>

```go
N9K01# ping 172.30.30.1 source-interface loopback 101
PING 172.30.30.1 (172.30.30.1): 56 data bytes
64 bytes from 172.30.30.1: icmp_seq=0 ttl=254 time=16.409 ms
64 bytes from 172.30.30.1: icmp_seq=1 ttl=254 time=2.995 ms
64 bytes from 172.30.30.1: icmp_seq=2 ttl=254 time=2.856 ms
64 bytes from 172.30.30.1: icmp_seq=3 ttl=254 time=2.728 ms
64 bytes from 172.30.30.1: icmp_seq=4 ttl=254 time=2.768 ms

--- 172.30.30.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 2.728/5.551/16.409 ms

N9K01# ping 172.31.31.1 source-interface loopback 101

PING 172.31.31.1 (172.31.31.1): 56 data bytes
36 bytes from 172.20.20.1: Destination Net Unreachable
Request 0 timed out
36 bytes from 172.20.20.1: Destination Net Unreachable
Request 1 timed out
36 bytes from 172.20.20.1: Destination Net Unreachable
Request 2 timed out
36 bytes from 172.20.20.1: Destination Net Unreachable
Request 3 timed out
36 bytes from 172.20.20.1: Destination Net Unreachable
Request 4 timed out

--- 172.31.31.1 ping statistics ---
5 packets transmitted, 0 packets received, 100.00% packet loss

N9K01# ping 172.31.31.1 source-interface loopback 102

PING 172.31.31.1 (172.31.31.1): 56 data bytes
64 bytes from 172.31.31.1: icmp_seq=0 ttl=254 time=3.784 ms
64 bytes from 172.31.31.1: icmp_seq=1 ttl=254 time=2.852 ms
64 bytes from 172.31.31.1: icmp_seq=2 ttl=254 time=2.133 ms
64 bytes from 172.31.31.1: icmp_seq=3 ttl=254 time=1.635 ms
64 bytes from 172.31.31.1: icmp_seq=4 ttl=254 time=2.783 ms

--- 172.31.31.1 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 1.635/2.637/3.784 ms
```
</details> 