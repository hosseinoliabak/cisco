# Quality of Service (QoS):
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
* **Queuing-Congestion Management:**
  * Round-Robin Scheduling: works very well for data applications as
it guarantees a certain bandwidth to each queue.
  * Weighted round-robin Scheduling
    * Cisco uses Class-Based Weighted Fair Queuing (CBWFQ)
  * Low Latency Queuing: Voice traffic has to be sent immediately and
  should not wait
* **Shaping and Policing:** are used to limit the bit rate
  * Policers do so by discarding traffic. (Policing is often used by
  ISPs who must limit the bitrate of their customers.)
  * Shapers will hold packets in a queue, adding delay. This is done on
  the customer side. The shaper will queue messages, delaying them to a
  certain CIR rate.
* **Congestion Avoidance:** The congestion avoidance tool can randomly
drop packets, or we can configure it to give certain packets a different
treatment based on their marking.

# IP Precedence and DSCP Values

## Type of Service (TOS)
IP packets have a field called the Type of Service field (also known as
the TOS byte)

## Class of Service (COS)
It is found in the header of 802.1Q (layer 2). It’s used for Quality of
Service on trunk links