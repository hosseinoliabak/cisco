# Campus LAN design, Spine-Leaf Network Design

In this post I am going to talk about different network architecture: the traditional campus design, then Spine-Leaf (AKA Leaf-and-spine architecture) and what problems the spine-leaf network design addresses.

## Classic Three-Tier Architecture

You can have a flat network design with a large broadcast domain wherein all network equipment, PCs, printers, APs, connects to each other by too many switches. On top of that, imagine you daisy-chain switches in turn together which increases the port numbers but also at the same time increases diameter of network.

To simplify the network design as well as for scalability we moved to the phase of hierarchical three-tier network design. Those layers include, Access layer, Distribution layer, and Core layer.

* **Access layer:** Provides Layer 2 connectivity of PCs, servers, APs, phones, and printers to the network.
* **Distribution (aggregation) Layer:** This layer aggregates the access layers. Access layer terminate it layer 2 connectivity here. Optionally, this layer provides FHRP to the VLANs in the access layer. Distribution layer on the other hand, communicates with core layer over layer 3.
* **Core (backbone) Layer:** This high speed switching switches packets as past as possible.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235387738-dc9e1834-6ca9-490b-8c1b-e7ece8b413c4.png" alt="Classic Three-Tier Architecture">
  <figcaption>Figure 1: Classic Three-Tier Architecture</figcaption>
</figure><br />

There is more into this topic but this is not in the scope of CCIE Datacenter. But here are some drawbacks with traditional network desing.

* In STP-based network, when a link or a switch fails, the STP recalculates which impacts the convergence. Imagine, if the problem occurs at root bridge and new root needs to be elected.
* STP-based networks, blocks the ports to prevent the Layer-2 loop.
* Prune to traffic storm because there is no TTL field in L2 header.
* In STP-based network, all the traffic for a particular VLAN forwards to the root bridge of that VLAN which might be a suboptimal path.
* Dedicated network for LAN and a separate dedicated Fiber-Channel network for SAN.

## Spine-Leaf Architecture

In Data center network, traffic generally flows in three directions:

* Noth-South Traffic: the communication in and out of the datacenter. For example, the traditional web browsing from the Internet to your Datacenter.
* East-West Traffic: the communication between servers and/or various applications within the datacenter. this is greatly facilitated by virtualization.
* Inter-DC traffic: traffic between data centers.

The modern workload of Data center traffic is server-server unlike the traditional client-server traffic.

In 1953, Charles Clos came up with a concept (in BSTJ) which is multistage fabrics as the best mathematical way to interconnect two nodes from an ingress call to an egress call. Clos Spine-Leaf topology was then built on that mindset.

In graph theory, This topology is a complete bipartite graph. A bipartite graph consists of two sets of disjoint vertices (here in networking sets of Spines and sets of Leafs). No vertex in the same set connects to another. (No Spine connects to another spine, no leaf connects to another leaf).

We moved Layer 3 links down to the access layer in Clos topology.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235387900-2ff62971-d09d-4822-9b0b-f8d88acda48b.png" alt="Clos Spine-Leaf Network Architecture">
  <figcaption>Figure 2: Clos Spine-Leaf Network Architecture</figcaption>
</figure>&nbsp;<br />

Leaf-and-spine topology scales very well. With one switch, this topology can handle 10K-200K of 10Gibps access ports. Next, let’s talk about the main components of these topology.

### Spine-Leaf Switches

Previously, we have discussed about the transition from campus network to leaf-and-spine architecture. Now, let’s see what hardware provide these functions.

* Leaf or top-of-rack (ToR) switch provides the connectivity to the servers. The upstream interfaces are layer-3 and the downstream towards the servers is a layer 2 classical ethernet.
* Spine (Backbone): This is the seconds stage in the fabric where leafs connect to.