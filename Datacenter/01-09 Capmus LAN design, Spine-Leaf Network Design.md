# Traditional and Modern Data Center Network Architecture

In this post I am going to talk about different network architecture: the traditional then Spine-Leaf (AKA Leaf-and-spine) architecture and what problems the spine-leaf network design addresses.

## Classic Three-Tier Architecture

You can have a flat network design with a large broadcast domain wherein all network equipment, PCs, printers, APs, connects to each other by too many switches. On top of that, imagine you daisy-chain switches in turn together which increases the port numbers but also at the same time increases diameter of network. But this not how we design an enterprise or a data center network.

Legacy data center networks are often hierarchical:
  * Consists of Access, Aggregation, and Core layers
  * Benefits of hierarchical network design include:
    * Modularity: facilitating change, but is lacking scalability
    * The ability to isolate faults in a specific layer can simplify troubleshooting efforts

* **Access layer:** Provides Layer 2 connectivity of PCs, servers, APs, phones, and printers to the network.
* **Distribution (aggregation) Layer:** This layer aggregates the access layers. Access layer terminate it layer 2 connectivity here. Optionally, this layer provides FHRP to the VLANs in the access layer. Distribution layer on the other hand, communicates with core layer over layer 3.
* **Core (backbone) Layer:** This high speed switching switches packets as past as possible.


<figure>
  <img src="https://github.com/hosseinoliabak/cisco/assets/31813625/cdc4e68d-12a6-48f6-8d72-5d3441e656dd" alt="Classic Three-Tier Architecture" /><br />
  <figcaption>Figure 1: Classic Three-Tier Architecture</figcaption>
</figure>

There is more into this topic but this is not in the scope of CCIE Datacenter. But here are some drawbacks with traditional network desing.

* Not all data paths are used.
  * Almost half of the links are unused.
  * Imagine thousand of fibre connections are not in use.
    * Not only the fibre itself, but two tranceivers per connection is unused.
* In STP-based network, when a link or a switch fails, the STP recalculates which impacts the convergence. Imagine, if the problem occurs at root bridge and new root needs to be elected.
* STP-based networks, blocks the ports to prevent the Layer-2 loop.
* Prune to traffic storm because there is no TTL field in L2 header.
* In STP-based network, all the traffic for a particular VLAN forwards to the root bridge of that VLAN which might be a suboptimal path.
  * Increased latency
* Multiple protocols used for inter-connectivity.
* Inefficient resource and power usage.
* Dedicated network for LAN and a separate dedicated Fiber-Channel network for SAN.

## Modern Architecture

In Data center network, traffic generally flows in three directions:

* Noth-South Traffic: the communication in and out of the datacenter. For example, the traditional web browsing from the Internet to your Datacenter.
* East-West Traffic: the communication between servers and/or various applications within the datacenter. this is greatly facilitated by virtualization.
* Inter-DC traffic: traffic between data centers.

The modern workload of Data center traffic is server-server unlike the traditional client-server traffic. The Applications drive changes:
  * Modern apploication flows
    * Increased VM-to-VM traffic
    * Additional east-west traffic
   * Requires more flat topology
  * Network virtualization
    * Intoroduces network overlays
    * Requires Physical-to-virtual integration for bare-metal servers
    * Requires overlay visualization and management
  * Everything as-a-service
    * Must be easy to scale out and scale back as demands change
    * Introduces multitenancy

### IP Fabric Infrastructure

IP Fabric is based entirely on IP infrastructure

In 1953, Charles Clos came up with a concept (in BSTJ) which is non-blocking, multistage, telephone switching architecture as the best mathematical way to interconnect two nodes from an ingress call to an egress call. Clos Spine-Leaf topology was then built on that mindset.

In graph theory, This topology is a complete bipartite graph. A bipartite graph consists of two sets of disjoint vertices (here in networking sets of Spines and sets of Leafs). No vertex in the same set connects to another. (No Spine connects to another spine, no leaf connects to another leaf).

We moved Layer 3 links down to the access layer in Clos topology.

You designate the ingress and egress crossbar switches as leaf nodes, while the middle-stage crossbar switches are referred to as spine nodes. Each access port on a leaf is precisely three nodes away from any other access port on a different leaf. This characteristic is what gives the Clos network its name as a three-stage fabric. Within a leaf-and-spine architecture, the objective is to distribute traffic across numerous paths throughout the fabric in order to achieve optimal traffic sharing.

<figure>
  <img src="https://github.com/hosseinoliabak/cisco/assets/31813625/69f29c87-d6a9-40d9-8a59-8ed3b3269af9" alt="Clos Spine-Leaf Network Architecture" /> <br />
  <figcaption>Figure 2: Clos Spine-Leaf Network Architecture</figcaption>
</figure>

<p>Leaf-and-spine topology scales very well. With one switch, this topology can handle 10K-200K of 10Gibps access ports. Next, let’s talk about the main components of these topology.</p>

### Five-stage fabric topology

<figure>
  <img src="https://github.com/hosseinoliabak/cisco/assets/31813625/ce2b5324-2286-461a-bf4a-96cbdf78e06e" alt="Figure 3: Five-Stage Architecture" /><br />
  <figcaption>Figure 3: Five-Stage Architecture</figcaption>
</figure>

### Spine-Leaf Switches

Previously, we have discussed about the transition from campus network to leaf-and-spine architecture. Now, let’s see what hardware provide these functions.

* Leaf or top-of-rack (ToR) switch provides the connectivity to the servers. The upstream interfaces are layer-3 and the downstream towards the servers is a layer 2 classical ethernet.
* Spine (Backbone): This is the seconds stage in the fabric where leafs connect to.
* All spines must adhere to a single model, while all leafs must also conform to a uniform model. Nevertheless, it is imperative that the models assigned to leafs and spines remain distinct from one another. :)

### Underlay and Overlay

* The Underlay
  * IP fabric is also known as the underlay.
  * The underlay consists of multiple equal cost L3 routed links that run between L2 endpoints.
  * The IP fabric can be combination of routing protocols.
    * OSPF is not routing protocol of choice for the fabric
    * Choose either eBGP or IS-IS
* The Overlay
  * Virtualizes L2 VLANs to carry Etherney frames across IP fabric
  * Two very critical technologies that compose the overlay are:
    * VXLAN - Encapsulates the frame in IP/UDP at the arriving L2 switchport
    * EVPN - Runs as part of the overlay to coordinate VXLAN endpoint