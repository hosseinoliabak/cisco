# IP Multicast Overview

Data delivery models
Let’s review what Multicast is by first looking into three other types of data delivery.

* Unicast: one-to-one transmission
* Broadcast: one-to-all transmission
* Anycast: one-to-nearest transmission

Multicast is data transmission from one or a group of publishers to a group of subscribers. With that being said we will have different applications with IP Multicast (IETF RFC-3170). An IP multicast application is any application which sends/receives data to/from IP multicast address (224.0.0.0/4).

### IP Multicast Application Taxonomy

* One-to-Many (1toM)
  * Single host transmits traffic to many receivers
    * IP Television
    * VoIP Music-on-hold (MOH)
    * Weather updates
    * News headlines
    * Sports scores
    * Network time
    * Hello packets
    * Stock prices
    * Sensor equipment
    * Security system
* Many-to-Many (MtoM)
  * Many hosts transmit/receive to same multicast group address
    * Multimedia conferencing
    * Concurrent processing
    * Shared document editing
    * Distance learning
    * Chat groups
    * Multiplayer games
* Many-to-one (Mto1)
  * Many hosts send data back to a source
    * Resource Discovery
    * Monitoring applications
    * Video surveillance
    * Auctions
    * Polling
    * Jukebox
    * Accounting

### Why not just unicast?

* Source must know the address of the destinations.
* Head-end replication: Source must generate one packet per destination.
* Bandwidth usage increases.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235561042-747c3106-6d30-4636-b325-8b28b45d73a9.png" alt="Unicast Forwarding">
  <figcaption>Figure 1: Unicast Forwarding</figcaption>
</figure>


**So, what is the advantage of Multicast, again?**

* The publisher does not need to know who the subscriber is. It’s the job of the network to find the subscribers.
* Publisher transmits a single feed for all subscribers.
* Only one packet replicate per interface, saving the bandwidth.
* Non-subscribers do not receive traffic.

<figure>
  <img src="https://user-images.githubusercontent.com/31813625/235561218-cba5347b-3985-4116-aa19-715a85739c9b.png" alt="Multicast Forwarding">
  <figcaption>Figure 2: Multicast Forwarding</figcaption>
</figure>


#### Multicast disadvantages:

* IP Multicast is UDP which is connectionless
  * No acknowledgement (best effort delivery)
  * No congestion avoidance
  * possible out-of-order packets

**Why not just use broadcast in LAN?** In that case, all end-hosts process all packets even if they don’t want them – which is a burden on receiver’s application layer.

## Common Multicast Components

Some of the common components used in multicast are:
  * Source
    * A multicast source (or publisher) is any device that originates multicast IP packets
  * MulticastIP Packet
    * Any IP packet that is destined to a multicast group address
    * The same packet should have a unicast source address
  * Group Address
    * An IP address in the range of 224.0.0.0/4
  * Receivers
    * A multicast receiver (or subscriber) is any host that is interested in receiving multicast IP packets
    * A Receiver uses IGMP to inform the directly connected router of its desire to receive multicast IP packets
  * Designated Router:
    * DR is the router closest to the source that forwards multicast packets into the network
    * If two or more routers are attached to the source, only one router becomes the DR on an election algorithm
  * Group Membership Protocol
    * Subscribers use group membership protocol to inform the directly connected router of their interest in receiving packets
    * Internet Group Management Protocol (IGMP) is used by IPv4 hosts and routers for this purpose
    * Multicast Listener Discovery (MLD) is used by IPv6 hosts and routers for this purpose
  * Multicast Routing Protocol
    * A multicast routing protocol is used between routers to build and maintain the multicast forwarding trees between the source and the receiver
    * The most common multicast routing protocol is Protocol Independent Multicast (PIM)

### How IP Multicast Works

Here is the flow of traffic from sender tot he receiver.

* Publisher sends UDP multicast traffic with “group” destination address.
  * Publisher sends traffic to destination address of the group.
* Subscribers “join” group address by signaling the router(s) on their LAN.
  * Subscribers listen for traffic going to group address.
* Routers between the source and the detitanation communicate to each other to build a loop-free tree.
* Groups use both layer 3 (routing) and layer 2 (switching) addresses.

# IPv4 Class D Addresses

Former IPv4 Class D address space (224.0.0.0/4) is reserved for multicast purposes (Steve Deering RFC 1112). There are some spaces reserved in this range. The blue and orange sections of the image below show which ranges are available for you to use.

Next, table below illustrates some of L3 and L2 multicast addresses which are familiar with you.

| Protocol    | L2 Address                             | L3 Address             |
| ----------- | -------------------------------------- | ---------------------- |
| OSPF        | 01-00-5E-00-00-05<br>01-00-5E-00-00-06 | 224.0.0.5<br>224.0.0.6 |
| All hosts   | 01-00-5E-00-00-01                      | 224.0.0.1              |
| All routers | 01-00-5E-00-00-02                      | 224.0.0.2              |
| VRRP        | 00-00-5E-00-01-XX                      | 224.0.0.18             |
| HSRPv2      | 00-00-0C-9F-FX-XX                      | 224.0.0.102            |
| IGMPv3      |                                        | 224.0.0.22             |

*Table 1: Multicast IPv4 and MAC Addresses* 

### Layer 2 Multicast Addressing

Multicast reserves MAC address range 01-00-5E-00-00-00 to 01-00-5E-7F-FF-FF (although it would be nice if whole OUI 01:00:5E was reserved for Multicast). Considering fixed first 25 bits of MAC address, the last 23 bits of MAC address maps to last 23 bits of an IPv4 address.

You must note that this implies that we have overlap between IPv4 to MAC address mapping. For example, 224.1.1.1, 224.129.1.1 have the same MAC address. Nevertheless let’s see the examples below for the conversion:

| IPv4                                | MAC Address       |
| ----------------------------------- | ----------------- |
| 230.255.1.2                         | 01-00-5e-7f-01-02 |
| 224.0.0.1<br>225.0.0.1<br>226.0.0.1 | 01-00-5e-00-00-01 |
| 239.1.1.1                           | 01-00-5e-01-01-01 |
| 232.255.254.253                     | 01-00-5e-7f-fe-fd |

*Table 2: IPv4 to Multicast MAC address conversion*