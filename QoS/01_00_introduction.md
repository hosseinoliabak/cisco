# Quality of Service (QoS):
Internet is best effort of FIFO (First In First Out). This is not QoS.

QoS controls help you better manage available bandwidth. QoS is about
using tools to change how the router or switch deals with different
packets. For example, we can configure the router so that voice traffic
is prioritized before data traffic.

## Characteristics of network traffic
There are four characteristics of network traffic that we must deal with:
* Bandwidth: the speed of the link. bits per second (bps)
* Delay: we have one-way (time from source to destination) and two-way delay (from source to destination and back)
  * Processing delay: the time for a device to perform the tasks
  * Queuing delay: the time that a packet is waiting in the queue
  * Serialization delay: the time that takes to send all bits of a frame to the PHY for transmission
  * Propagation delay: the time it takes for bits to cross the PHY. We cannot change this type of delay.
* Jitter: variation of one-way delay. The receiver of packets (for voice experience)
must deal with jitter, making sure the packets have a steady delay. 
* Loss: usually as a percentage of lost packets sent. When there is a congestion,
packets will be queued but once the the queue is ful, packets will be dropped. With QoS
we can decide which packets drop when this happens.  

## Traffic types
What we need to configure depends on the application that we use.
* Batch application: download a file from the Internet.
  * Bandwidth is nice to have
  * There is a one-way delay which is not that much important
  * Packet loss: Because file transfer uses TCP connection to retransmit the data, so
  the packet loss doesn't matter. 
* Interactive application such as SSH
  *  Does't require a lot of bandwidth
  * Sensitive to delay and packet loss
* Voice and video
  * Very sensitive to delay, jitter, and packet loss
  * For voice traffic we need
    * One-way delay < 150 ms
    * Jitter < 30 ms
    * Loss < 1%
  * For interactive video traffic we have this guide:
    * One-way delay" 200-400 ms
    * Jitter: 30-50 ms
    * Loss: 0.1%-1% 

## QoS Tools
* **Classification and marking:** Before we can give certain packets a
better treatment, we first have to identify those packets. This is
called **classification**. Once the traffic is classified, it’s best
practice to **mark the packet**. The reason that we use marking is that sometimes the 
classification requires some complex ACL rules and can degrade performance on the router
or switch that is doing classification. They swill still do classification but only has
to look for marked packets. Most IP phones mark IP packets that they are create.
Another useful tool from classification is NBAR (Network-Based Application Recognition).
NBAR is able to detect applications from your network traffic ans is able to do so by
looking at the content of IP packet.
Marking means we <u>change one or more of the header fields</u> in a packet (DSCP) or frame. For example, an <i> IP
packet</i> has the ToS (Type of Service) field that we can use to mark
the packet.
<i>Ethernet frames</i> don’t have such field but we do have something for
trunks. The tag that is added by 802.1Q has a priority field.

* **QoS Treatment:** pure default device configurations usually don't provide any special treatment
  * **Queuing and scheduling:** provides congestion management. Packets are queued during congestion to prevent drops (tail-drops).
    * FIFO-Default: default on routers and switches
    * For IOS switch
      * Round-Robin Scheduling (standard queuing method used by switches): Cycles through the queues in order, taking turns with each queue.
      Works very well for data applications as it guarantees a certain bandwidth to each queue.
      * Priority queuing: Strict priority queuing for switches.
    * For IOS XE router and switch:
      * Weighted round-robin Scheduling (specially on routers)
        * Cisco uses Class-Based Weighted Fair Queuing (CBWFQ): guaranties a minimum bandwidth
        to each class when there is congestion.
      * Low Latency Queuing: Voice traffic has to be sent immediately and should not wait.
        * Priority queue for CBWFQ
        * Setting a limit to the priority queue introduce another problem. What if we have 
        so much voice traffic? This is usually solved by Call Admission Control (CAC). CAC
        is something we configure on our PBX which ensure that you can only have X amount
        of voice calls simultaneously.
  * **Shaping and Policing:** are used to limit the bit rate
    * Policing: do so by discarding traffic. (Policing is often used by
    ISPs who must limit the bitrate of their customers to Committed Information Rate (CIR).)
      * Drops exceeding traffic
      * Use when you are trying to limit a specific traffic type from stealing all of the bandwidth
      * Inbound or outbound
      * For example we can use policing for Windows update to be dropped if it is stealing our bandwidth.
    * Shaping: will hold packets in a queue, adding delay. This is done on
    the customer side. The shaper will queue messages, delaying them to a
    certain CIR rate.
      * Buffers the exceeding traffic
      * Use when you are trying to match the bandwidth on a WAN link. For example you 
      have a 100Mi interface but your CIR is 20Mi. Shaping makes sure that the traffic rate
      you are sending to the provider matches what they are limiting you to.
      * Only outbound
  * **Congestion Avoidance:** The congestion avoidance tool can randomly
  drop packets from output queue, or we can configure it to give certain packets a different
  treatment based on their marking.
     
## Type of Service (TOS)
IP packets have a field called the Type of Service field (TOS byte)

### IPv4 Packet Header
```
   0               1               2               3               4
   0 1 2 3 4 5 6 7 8             15 16                            31
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+     ^
   |Version|  IHL  |Type of Service|          Total Length         |     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+     |
   |         Identification        |Flags|      Fragment Offset    |     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+     |
   |  Time to Live |    Protocol   |         Header Checksum       |  20 Bytes
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+     |
   |                       Source Address                          |     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+     |
   |                    Destination Address                        |     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+     -
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   |                             Data                              |
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
* **Type of Service byte/Differentiate Services field (8 bits):** 
  * In 1981 (RFC 791)
    * 3 bits Precedence: the higher the value, the more important the IP packet is
    * 5 bits type of service: used to assign what kind of delay, throughput,
    and reliability we want. Never really been used.
  * In 1992 (RFC 1349)
    * 3 bits Precedence
    * 4 bits Type of Service: Never been used!
    * 1 bit Must be zero (MBZ)
  * In 1998 (RFC 1474): TOS byte got a new name (Differentiate Services or DS field)
    * DSCP (6 bits): used to set a *codepoint* that will affect the Per Hop Behaviour (PBH)
    at each node. It is like Precedence in TOS byte to set a certain priority.
      * You can use either
        * Default
        * decimal values between 0-63
        * CS values between 0-7
        * Assured forwarding
        
        | | Class 1 | Class 2 | Class 3 | Class 4 |
        | --- | --- | --- | --- | --- |
        | Low | AF11 | AF21 | AF31 | AF41 |
        | Med | AF12 | AF22 | AF32 | AF42 |
        | High | AF13 | AF23 | AF33 | AF43 |
      * DSCP EF of DSCP 46 (Expedited Forwarding) is normally used for voice traffic
      * DCSP CS3 or AF31 is used for call assigning
    * CU (2 bits): there 2 bits are currently unused and ignored
    
### Class of Service (COS) - 802.1p
It is found in the header of 802.1Q (layer 2). It’s used for Quality of
Service on trunk links. (Priority field on 802.1Q tag).
  * Values 0-7: higher value, higher priority.

### QoS Marking best practice

| Traffic | COS | Old-schoo l DSCP | DSCP | 
| --- | --- | --- | --- | 
| Routing | 6 | CS6 | 48 |
| Voice | 5 | EF | 46 |
| Interactive-video | 4 | AF41 | 34 |
| Streaming-video | 4 | CS4 | 32 |
| Call-signaling | 3 | AF31/CS3 | 26/24 |
| Management | 2 | CS2 | 16 |
| Best effort | 0 | 0 | 0 |
 