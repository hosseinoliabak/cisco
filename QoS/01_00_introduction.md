# Introduction to Quality of Service (QoS):
Internet is best effort of FIFO (First In First Out). This is not QoS.

QoS controls help you better manage available bandwidth. QoS is about
using tools to change how the router or switch deals with different
packets. For example, we can configure the router so that voice traffic
is prioritized before data traffic.

## Characteristics of network traffic
There are four characteristics of network traffic that we must deal with:
* Bandwidth
* Delay
  * Processing delay
  * Queuing delay
  * Serialization delay
  * Propagation delay
* Jitter
* Loss

## QoS Tools:
* **Classification and marking:** Before we can give certain packets a
better treatment, we first have to identify those packets. This is
called **classification**. Once the traffic is classified, it’s best
practice to **mark the packet**. Marking means we <u>change one or more
of the header fields</u> in a packet or frame. For example, an <i> IP
packet</i> has the ToS (Type of Service) field that we can use to mark
the packet.
<i>Ethernet frames</i> don’t have such field but we do have something for
trunks. The tag that is added by 802.1Q has a priority field:
  * By ACL: degrades the performance on the router or switch
  * Most IP phones mark IP packets that they create
  * Network-Based Application Recognition (NBAR): By looking at the
   content of IP packets
* **Queuing-Congestion Management**
  * Round-Robin Scheduling: works very well for data applications as
it guarantees a certain bandwidth to each queue.
  * Weighted round-robin Scheduling
    * Cisco uses Class-Based Weighted Fair Queuing (CBWFQ)
  * Low Latency Queuing

* Shaping and Policing
* Congestion Avoidance