# Multicast Distribution Tree

A multicast distribution tree (MDT) is the paths through the network from source to receive used to distribute multicast data traffic. There are two types of trees:

  * Shortest Path Tree (SPT): this tree is also known as source tree which is the quickest path from the source to receiver. But it may require more states in the network.
  * Shared Tree (RPT): this tree uses SPT from each sender to a shared root which is known as the Rendezvous Point (RP), then it creates an SPT from RP to receiver.

Now it is time to talk about some jargons in multicast, then we will continue speaking on different types of trees.


  * **First Hop Router (FHR):** The gateway of the multicast source/publisher.
  * **Last Hop Router (LHR):** The gateway of a receiver/subscriber.
  * **Reverse Path Forwarding (RPF) Check:** When a packet arrives to the route, before forwarding it, router checks if the source IP address is in it’s routing table on that interface (strict RPF check), in that case RPF check succeeds. If the source IP is not on the interface were packet arrives to the router, RPF check fails and packet is discarded. RPF check here helps prevent loops and floods.
  * **Incoming interface (IIF):** This is your RPF interface – The interface towards the multicast source with best IGP metric/cost. If multiple interfaces have the same cost, the interface with the highest IP address is chosen as the tiebreaker.
  * **Outgoing interface list (OIL):** This interface faces the receiver. When a new receiver joins the group, the OIL updates.


<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235804011-bd6145b0-6522-42cb-a2cf-188593ee8adb.png" alt="Multicast Terminologies">
  <figcaption>Figure 1: Multicast Terminologies</figcaption>
</figure>


### Shortest Path Tree (SPT)

With shortest path tree or the source tree, traffic flows from the source and FHR (the root of the tree) to the receivers (the leaves) via the shortest path (that is closest to the receivers). Multicast forwarding table represents the source tree by (S, G). S indicates the Source address and G indicates group address.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235806165-ce35dd6c-b0ef-4954-ac70-490748ca833e.png" alt="Distinct Source Trees">
  <figcaption>Figure 2: Distinct Source Trees</figcaption>
</figure>


With SPT, each router in the path must share and maintain state information. It would be very difficult for every router to manage this process completely independently in a very large network, or if there were a great number of sources. That is why the IETF introduced another type of network tree: the shared tree.

Summary:
  * Source tree represents by (S,G)
  * SPT roots at FHR for (S,G)

### Shared Tree or Rendezvous-Point Tree (RPT)

Shared trees shift the initial tree-building process to a single router known as a rendezvous point (the root of shared trees). RP is the router where the two trees meet each other: FHR sends the multicast traffic to RP. LHR goes to RP to receive the traffic.

In the ASM model, the receiver subscribe to a group using IGMP. In the subscription request, they do not indicate the source. The routers must build a tree toward an unknown source. The solution to this problem is to have a meeting point for sources and receiver in the network. This meeting point is called an RP.

Multicast Routing Information Base (MRIB) and MFIB represent the shared tree as (*, G). (*,G) information is shared upstream (because it's shared by all receivers) from LHRs to the RP. Each (*,G) is a single tree regardless of the source location. This means that only one tree is required for the group, even if there are many sources. The tree from FHR to the RP is (S,G).

The drawback of shared trees is that the subscribers to the same multicast group receive traffic from all the sources publishing the traffic to that group.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235806357-69738839-e5e7-4b35-b62c-853de7dd6b18.png" alt="Shared Tree (RPT)">
  <figcaption>Figure 3: Shared Tree (RPT)</figcaption>
</figure>



Summary:
  * RPT roots at RP for (*,G)
  * There are two trees. (S,G) from the source to RP. and (*,G) from receivers to RP.

### Bidirectional Shared Tree

Bidirectional shared tree scales very well with M-to-M applications. Let’s say you have 100 hosts part of videoconferencing that are sending traffic to one group address; with RPT, you would require 100x(S,G)+1x(*,G) = 101 entry on RP.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235806530-9896c35f-916c-4843-afab-8b926a33421d.png" alt="Bidirectional shared tree">
  <figcaption>Figure 4: Bidirectional shared tree</figcaption>
</figure>

With bidirectional shared-tree, multicast groups are carried across the network over bidirectional shared trees, hence we never would have the (S,G) entry. We only have wildcard-source (*,G) routes. So, for the example above, we only need one (*,G) which is rooted on RP. With bidirectional shared tree, traffic can flow on both directions to and from the sources for each group. Loop prevention in bidirectional tree is different than RPF check. We will use Designated Forwarder for this purpose. With designated forwarder, only one router in each link (including point-to-point links) can forward the multicast traffic. DF accepts data on its OIL then sends out all other interfaces including IIF.