# Underlay Transport for VXLAN

In the first section of our VXLAN course, we talked about overlay, underlay, and some terminologies associated with this technology. By now, you know what an underlay is.

Within our datacenter context, the underlay is our Clos network which includes the leafs and spines and their interconnections. To get rid of STP, we are running L3 links between leafs and spines. Typically, the reachability distributes via a routing protocol.

We also discussed that your underlay must consider 50-54 bytes of VXLAN header. So, make sure you adjust MTU according to this consideration.

In the following, I am going more in-depth with regard to the underlay consideration.

### IP Addressing

In this section, let’s talk about IP addressing in our VXLAN Clos network architecture throughout this course. The table below is just to give you an idea of what I recommend for addressing:

| **Interface** | **Node**         | **Purpose**                              |
|---------------|------------------|------------------------------------------|
| Loopback0     | Leafs and Spines | Unnumbered interfaces, IGP RID, BGP, PIM |
| Loopback1     | Leafs            | NVE interface                            |
| Loopback254   | Spines           | PIM Anycast RP                           |

*Table 1: High level summary of the IP Plan*

We will talk about Loopback254 later in another post.


##### Detailed IP Plan:

| Node      | ISIS (NSAP)                                 | L0 ( BGP, PIM) | Multicast Anycast L254 | MultiSite Anycast IP L2 | Lo1 (NVE)     | vPC Anycast VTEP<br>Lo1 Secondary | vPC Keepalive<br>(MGMT) | VLAN 100 (VNI 20100)<br>SMENODE | VLAN 200 (VNI 20200)<br>SMENODE | L3VNI 23967<br>VRF SMENODE | Border Link                                          |
| --------- | ------------------------------------------- | -------------- | ---------------------- | ----------------------- | ------------- | --------------------------------- | ----------------------- | ------------------------------- | ------------------------------- | -------------------------- | ---------------------------------------------------- |
| T-SPINE01 | net 49.0000.0001.0001.00<br>is-type level-2 | 10.0.0.1/32    | 10.0.0.254/32          | \-                      | \-            | \-                                | 10.0.11.1/24            | \-                              | \-                              | \-                         |                                                      |
| T-SPINE02 | net 49.0000.0001.0002.00<br>is-type level-2 | 10.0.0.2/32    | 10.0.0.254/32          | \-                      | \-            | \-                                | 10.0.11.2/24            | \-                              | \-                              | \-                         |                                                      |
| T-LEAF01  | net 49.0000.0001.1001.00<br>is-type level-2 | 10.0.1.1/32    | \-                     | \-                      | 10.0.5.1/32   | 10.0.9.1/32                       | 10.0.12.1/24            | 10.0.100.254/24                 | 10.0.200.254/24                 | \-                         |                                                      |
| T-LEAF02  | net 49.0000.0001.1002.00<br>is-type level-2 | 10.0.1.2/32    | \-                     | \-                      | 10.0.5.2/32   | 10.0.9.1/32                       | 10.0.12.2/24            | 10.0.100.254/24                 | 10.0.200.254/24                 | \-                         |                                                      |
| T-LEAF03  | net 49.0000.0001.1003.00<br>is-type level-2 | 10.0.1.3/32    | \-                     | \-                      | 10.0.5.3/32   | 10.0.9.2/32                       | 10.0.12.3/24            | 10.0.100.254/24                 | 10.0.200.254/24                 | \-                         |                                                      |
| T-LEAF04  | net 49.0000.0001.1004.00<br>is-type level-2 | 10.0.1.4/32    | \-                     | \-                      | 10.0.5.4/32   | 10.0.9.2/32                       | 10.0.12.4/24            | 10.0.100.254/24                 | 10.0.200.254/24                 | \-                         |                                                      |
| T-BGW01   | net 49.0000.0001.2001.00<br>is-type level-2 | 10.0.4.252/32  | \-                     | 10.0.4.254.254/32       | 10.0.8.252/32 | 10.0.10.252/32                    | 10.0.12.5/24            | \-                              | \-                              | \-                         | 172.16.1.0/29 (AS-64600)<br>172.18.1.0/29 (AS-64800) |
| T-BGW02   | net 49.0000.0001.2002.00<br>is-type level-2 | 10.0.4.253/32  | \-                     | 10.0.4.254.254/32       | 10.0.8.253/32 | 10.0.10.252/32                    | 10.0.12.6/24            | \-                              | \-                              | \-                         | 172.16.1.8/29 (AS-64600)<br>172.18.1.8/29 (AS-64800) |
| SRV1      |                                             | \-             | \-                     |                         | \-            |                                   |                         | 10.0.100.11/24 GW: .254         | \-                              | \-                         |                                                      |
| SRV2      |                                             | \-             | \-                     |                         | \-            |                                   |                         | \-                              | 10.0.200.12/24 GW: .254         | \-                         |                                                      |
| SRV3      |                                             | \-             | \-                     |                         | \-            |                                   |                         | 10.0.100.13/24 GW: .254         | \-                              | \-                         |                                                      |
| SRV4      |                                             | \-             | \-                     |                         | \-            |                                   |                         | \-                              | 10.0.200.14/24 GW: .254         | \-                         |                                                      |
| SRV5      |                                             | \-             | \-                     |                         | \-            |                                   |                         | 10.0.100.15/24 GW: .254         | \-                              | \-                         |                                                      |

*Table 2: Detailed IP Plan*

##### Summary of the IP Plan:

|           **DC-1**             | **10.0.0.0/14** |                                         |
| --------------------------------- | --------------- | --------------------------------------- |
| maximum number of spines          | 254             | 10.0.0.0/24                             |
| maximum unnnumbered IPs for leafs | 1024            | 10.0.1.0/24 + 10.0.2.0/23 + 10.0.4.0/24 |
| maximum number of VTEPs + BLEAFs  | 1024            | 10.0.1.0/24 + 10.0.2.0/23 + 10.0.4.0/24 |
| anycast VTEPs needed              | 512             | 10.0.9.0/24 + 10.0.10.0/24              |

*Table 3: Summary of the IP Plan*

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

NVE interface (which is your tunnel/overlay interface) has to get its IP address from a loopback interface which is always available. From a design perspective, separating underlay from overlay function is desirable. So, we are not going to use the loopback interface which we created for our underlay.

```c
LEAF01(config-vlan)#interface loopback1
LEAF01(config-if)#description VXLAN-VTEP-NVE-IF
LEAF01(config-if)#ip address 192.168.250.1/32
LEAF01(config-if)#ip router isis UNDERLAY
```
### OSPF as the Underlay Routing Protocol

The default OSPF network type for Ethernet interfaces is broadcast. Since we are only having two points on each side of the link, we will change the interface type to point-to-point to avoid DR/BDR election process.

In order to optimize SPF calculation, we issue the `ispf` command (to enable incremental SPF) under the OSPF process. When an interface which does not belong to SPT (stub network or a transit link not participating in SPT) goes down, instead of running full SPF, the switch runs incremental SPF. Nevertheless, when a link comes up, the full SPF must still run.

OSPF routers are identified by router-id which has nothing to do with area ID

### IS-IS as the Underlay Routing Protocol

IS-IS routers are identified by NET (NET Entity Title). Here is an example of a NET address:
  * 49.0002.1921.6802.4001.00
  * 49.0002.1921.6803.6001.00

| Component | Description                                                                      | Example                        |
| --------- | -------------------------------------------------------------------------------- | ------------------------------ |
| Area      | Can range from 1 to 13 Byte(s) in length                                         | 49.0002                        |
| System ID | Always 6 Bytes. Can be any set of hexadecimal digits                             | 1921.6802.4001 <br /> 1921.6803.6001 |
| Selector  | Identifies the destination network layer service that should receive the traffic | 00                             |

*Table 4: SIS-IS NSAP Addressing*

### BGP as the Underlay Routing Protocol

BGP as a hard state protocol sends updates only when there is a change in NRLI. Spines usually do not host VTEP interface; so, we set the next-hop attribute to unchanged. If you use BGP in your underlay, remember that you will be having two different address-families per leaf: ipv4 unicast for underlay and L2VPN EVPN for overlay. Also, if the leaf connects to MPLS, you would also require VPNv4 as the third address-family.

## Overlay

The NVO requires a mechanism to know which end hosts (identities) are behind which edge device (location). The Leaf switch builds a location-identity mapping database. There are three ways to facilitate the mapping.
  * Via Data Plane: through Flood and learn (F&L)
  * Via Overlay Control Plane: BGP EVPN
  * Via a central controller: Such as Cisco APIC or OpenDaylight

Note that in this context, under the umbrella of **Network Overlay**, the *leaf switch* pushes and pops the overlay headers. We can also have **Host Overlay** runs on a virtual switch or router on a physical server. Both network overlay and host overlays are very popular.