# VXLAN Improvement to Datacenter Fabric
In this section, I am going to talk about data center fabric journey and how Virtual eXtensible LAN emerged. I will start with STP, then vPC and PortChannel. I won’t talk about FabricPath. We will finally jump in to VXLAN in Clos networks.

### STP Drawbacks
Let’s architect our Clos topologies with L2 links. Then STP blocks some ports.
![vxlan-Page-3 drawio](https://user-images.githubusercontent.com/31813625/232259851-98834f21-0728-4858-882a-b6f22c2c7882.svg)
*STP Drawbacks*

Spanning-tree had some drawbacks for us which I am going to summarize them here:
* Convergence issue
  * Imagine with 100 Gibps link, a one-second convergence time can result in 12.5 GB of traffic loss.
* Unused links
* No multihoming
* Suboptimal forwarding
  * The root is on a single switch, all traffic forwards through that switch.
* No ECMP
* Traffic storm
  * Remember, there is no TTL in L2 header
* Scalability
  * Maximum of 4K VLANs; typically no multitenancy.


### STP to vPC improvement
vPC provides multihoming from an L2 perspective. You will also be having active-active forwarding with vPC. However, we don’t want run vPC between leafs and spines. We would very much like to run vPC between the leafs downstream to the servers.

![vxlan-vpc drawio-1](https://user-images.githubusercontent.com/31813625/232260139-dcc9f159-cff6-4a19-a248-8c92ccae4a11.svg)
*STP to vPC improvement*


### VXLAN Improvements
Virtual eXtensible LAN which is open standard addresses all the STP limitations.
* Data Plane: VXLAN RFC-7348
* Control Plane: EVPN MP-BGP RFC-7432
It supports up to 16M broadcast domains (as opposed to 4K limitation). With VXLAN you will have ECMP, because VXLAN runs over IP. It also allows for Multitenancy and includes scaling enhancements.

### What is VXLAN
Virtual eXtensible LAN is a Layer 2 overlay Data Plane technology. With that being said, you can stretch your Layer 2 Network over Layer 3 network. We talked about overlay which is the contrast of underlay. Here, I would very much like to give a brief explanation on these two terms:

* **Underlay:** This is your IP connectivity; your pure IP packet; without any additional header. Here the considerations for our underlay:
  * IGP with Equal Multipath
  * No Spanning-tree
  * Maybe multicast
* **Overlay:** This is a static or dynamic tunnel that runs on top of underlay. I mean you encapsulate overlay inside underlay. In the context of this post, your VXLAN is IP/UDP encapsulation.
  * An overlay in Datacenter is referred to as a network virtualization overlay which consists of:
    * **Identity:** Identifies the IP/MAC of the end host.
      * The inner header
    * **Location:** Identifies the tunnel edge device that is responsible for encapsulating and de-encapsulating the tunnel traffic.
      * The outer header
![Overlay](https://user-images.githubusercontent.com/31813625/232260297-1d6b26b5-daa9-4d9e-857f-7f88228cfc72.svg)
*Overlay*
I should note that overlays incur overhead of n bytes, where n is the size of the overlay header. With that being said, you must provision the underlay with proper MTU to ensure delivery of overlay traffic. With VXLAN, the overhead is typically 50B (or 54 Byte if you also consider 802.1Q tag). So, make sure you adjust the MTU to 9050 or 9054 bytes across all your L2 links.
![vxlan-vxlan-header drawio](https://user-images.githubusercontent.com/31813625/232260318-5c366b57-8ca3-4ec5-abd2-a54d64bc7593.svg)
*VXLAN Frame Format*

### Terminologies
To close this post and before we start diving into topic and the configuration, let’s talk about the terms which we will be discussing throughout the VXLAN course.

  * **Edge device:** This is typically your leaf which is responsible for encapsulating and de-encapsulating the traffic.
  * **VXLAN Tunnel Endpoint (VTEP) Interface:** (AKA Network Virtualization Edge (NVE) interface) is an interface on your edge device which participates in VXLAN. Only one NVE interface is allowed on the switch.
  * **VXLAN Network Identifier (VNI):** VNI is 3-Byte long which is analogous to VLAN ID in the VLAN segment.
  * **VXLAN core:** Interconnects different VTEPs. the core can be regular IP routers.
  * **VXLAN Segment:** The resulting L2 Overlay.
  * **Network Virtualization Overlay (NVO):** Isolates network traffic per tenant.
  * **Multidestination Traffic:** NVE needs to handle multidestination traffic types which are known as Broadcast, Multicast, and Unknown Unicast (BUM) traffic types in order to receive them from the overlay and then replicate and transport them in the underlay. There are two possible ways to handle multidestination traffic
    * **IP Multicast:** You need to configure IP Multicast in the underlay
    * **Ingress Replication:** Multiple unicast packets are sent to forward multidestination traffic.