# Underlay Transport for VXLAN
In the first section of our VXLAN course we talked about overlay, underlay, and some terminologies with this technology. By now, you know what an underlay is.

Within our datacenter context the underlay is our Clos network which includes the leafs and spines and their interconnections. To get rid of STP, we are running L3 links between leafs and spines. Typically, the reachability distributes via a routing protocol.

We also discussed that your underlay must consider 50-54 bytes of VXLAN header. So, make sure you adjust MTU according to this consideration.

In the following I am going more in depth with regard to the underlay consideration.

### IP Addressing
In this section Let’s talk about IP addressing in our VXLAN Clos network architecture throughout this course. The table below is just to give you an idea of what I recommend for addressing:

| **Interface** | **Node**         | **Purpose**                              |
|---------------|------------------|------------------------------------------|
| Loopback0     | Leafs and Spines | Unnumbered interfaces, IGP RID, BGP, PIM |
| Loopback1     | Leafs            | NVE interface                            |
| Loopback254   | Spines           | PIM Anycast RP                           |
*IP Addressing Summary*

We will talk about Loopback254 later in another post.

### Ethernet Interfaces between Leafs and Spines
Since the Cisco NX-OS switches support IP unnumbered, it is a good practice to create a loopback interface with an IP address that matches your IGP (IS-IS, OSPF) RID, then reuse that IP address for Ethernet interfaces.

```c
LEAF01#configure terminal 
LEAF01(config)#interface loopback 0
LEAF01(config-if)#ip add 192.168.1.1/32
LEAF01(config)#interface ethernet 1/1
LEAF01(config-if)#no switchport 
LEAF01(config-if)#medium p2p 
LEAF01(config-if)#ip unnumbered loopback 0
LEAF01(config-if)#mtu 9192
```
### NVE Interface IP Address
NVE interface (which is your tunnel/overlay interface) has to get its IP address from a loopback interface which is always available. From design perspective, separating underlay from overlay function is desirable. So, we are not going to use the loopback interface which we created for our underlay.

```c
LEAF01(config-vlan)#interface loopback1
LEAF01(config-if)#description VXLAN-VTEP-NVE-IF
LEAF01(config-if)#ip address 192.168.250.1/32
LEAF01(config-if)#ip router isis UNDERLAY
```
### OSPF as an Underlay
The default OSPF network type for ethernet interfaces is broadcast. Since we are only having two points on each side of the link we will change the interface type to point-to-point to avoid DR/BDR election process.

In order to optimize SPF calculation, we issue the ispf command (to enable incremental SPF) under the OSPF process. When an interface which does not belong to SPT (stub network or a transit link not participating in SPT) goes down, instead of running full SPF, the switch runs incremental SPF. Nevertheless, when a link comes up, the full SPF must still run.

### IS-IS as an Underlay
IS-IS routers are identified by NSAP address where the first 13 bits define the IS-IS area and the next 48 bits identify the router. (Remember that OSPF routers are identified by router-id which has nothing to do with area ID).

### BGP as an Underlay
BGP as a hard state protocol sends updates only when there is a change in NRLI. Spines usually do not host VTEP interface; so, we set the next-hop attribute to unchanged. If you use BGP in your underlay, remember that you will be having two different address-families per leaf: ipv4 unicast for underlay and L2VPN EVPN for overlay. Also if the leaf connects to MPLS, you would also require VPNv4 as the third address-family.

## Overlay

Then NVO requires a mechanism to know which end hosts (identities) are behind which edge device (Location). The Leaf switch builds a location-identity mapping database. There are three ways to facilitate the mapping.
  * Via Data Plane: through Flood and learn (F&L)
  * Via Overlay Control Plane: BGP EVPN
  * Via a central controller: Such as Cisco APIC or OpenDaylight

Note that in this context, I mention the leaf switch pushes and pops the overlay headers. But this is under the umbrella of network overlay. We can also have Host Overlay runs on a virtual switch or router on a physical server. Both network overlay and host overlays are very popular.