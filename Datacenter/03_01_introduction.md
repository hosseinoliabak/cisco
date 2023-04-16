# VXLAN Improvement to Datacenter Fabric

In this section, I am going to talk about the data center fabric journey and how Virtual eXtensible LAN emerged. I will start with STP, then vPC and PortChannel. I won't talk about FabricPath. Finally, we will jump into VXLAN in Clos networks.

### STP Drawbacks

Let's architect our Clos topologies with L2 links. Then STP blocks some ports.

![vxlan-Page-3 drawio](https://user-images.githubusercontent.com/31813625/232259851-98834f21-0728-4858-882a-b6f22c2c7882.svg)

*STP Drawbacks*

Spanning-tree had some drawbacks for us which I am going to summarize here:

* Convergence issue
  * Imagine with a 100 Gbps link, a one-second convergence time can result in 12.5 GB of traffic loss.
* Unused links
* No multihoming
* Suboptimal forwarding
  * The root is on a single switch, and all traffic forwards through that switch.
* No ECMP
* Traffic storm
  * Remember, there is no TTL in L2 header.
* Scalability
  * Maximum of 4K VLANs; typically no multitenancy.

### STP to vPC improvement

vPC provides multihoming from an L2 perspective. You will also have active-active forwarding with vPC. However, we don't want to run vPC between leafs and spines. We would very much like to run vPC between the leafs downstream to the servers.

![vxlan-vpc drawio-1](https://user-images.githubusercontent.com/31813625/232260139-dcc9f159-cff6-4a19-a248-8c92ccae4a11.svg)

*STP to vPC improvement*

### VXLAN Improvements

Virtual eXtensible LAN, which is an open standard, addresses all the STP limitations.

* Data Plane: VXLAN RFC-7348
* Control Plane: EVPN MP-BGP RFC-7432

It supports up to 16M broadcast domains (as opposed to the 4K limitation). With VXLAN, you will have ECMP because VXLAN runs over IP. It also allows for multitenancy and includes scaling enhancements.


### What is VXLAN

Virtual eXtensible LAN (VXLAN) is a Layer 2 overlay Data Plane technology that allows you to stretch your Layer 2 Network over a Layer 3 network. In contrast to the underlay, which is the IP connectivity without any additional header, overlay is a static or dynamic tunnel that runs on top of the underlay, encapsulating overlay inside underlay. In the context of this post, VXLAN is IP/UDP encapsulation.

- Underlay: This refers to the IP connectivity and pure IP packets without any additional header. Considerations for underlay include:
  - IGP with Equal Multipath
  - No Spanning-tree
  - Maybe multicast
- Overlay: This refers to a network virtualization overlay that consists of:
  - Identity: This identifies the IP/MAC of the end host and is represented by the inner header.
  - Location: This identifies the tunnel edge device that is responsible for encapsulating and de-encapsulating the tunnel traffic, represented by the outer header.

![Overlay](https://user-images.githubusercontent.com/31813625/232260)

*VXLAN Frame Format*

### Terminologies

To close this post and before we start diving into the topic and configuration, let’s talk about the terms which we will be discussing throughout the VXLAN course.

- **Edge device:** This is typically your leaf which is responsible for encapsulating and de-encapsulating the traffic.
- **VXLAN Tunnel Endpoint (VTEP) Interface:** (AKA Network Virtualization Edge (NVE) interface) is an interface on your edge device which participates in VXLAN. Only one NVE interface is allowed on the switch.
- **VXLAN Network Identifier (VNI):** VNI is a 3-Byte long identifier analogous to VLAN ID in the VLAN segment.
- **VXLAN core:** This interconnects different VTEPs and can be regular IP routers.
- **VXLAN Segment:** This represents the resulting L2 Overlay.
- **Network Virtualization Overlay (NVO):** This isolates network traffic per tenant.
- **Multidestination Traffic:** NVE needs to handle multidestination traffic types which are known as Broadcast, Multicast, and Unknown Unicast (BUM) traffic types in order to receive them from the overlay and then replicate and transport them in the underlay. There are two possible ways to handle multidestination traffic:
  - **IP Multicast:** You need to configure IP Multicast in the underlay.
  - **Ingress Replication:** Multiple unicast packets are sent to forward multidestination traffic.
