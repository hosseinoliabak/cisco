# QoS Implementation Tools

QoS Sequence:

![image](https://user-images.githubusercontent.com/31813625/40927338-7cb590b8-67ec-11e8-8a2e-93bb6ec4fb4d.png)


## Classification and marking tools
Before we can give certain packets a better treatment, we first have to identify those packets. This is
called **classification**. Once the traffic is classified, it’s best practice to **mark the packet** (happens in both L2 and L3).
The reason that we use marking is that sometimes the classification requires some complex ACL rules and can degrade
performance on the router or switch that is doing classification. They swill still do classification but only has
to look for marked packets. Most IP phones mark IP packets that they are create.
Another useful tool from classification is NBAR (Network-Based Application Recognition).
NBAR is able to detect applications from your network traffic ans is able to do so by
looking at the content of IP packet.
Marking means we <u>change one or more of the header fields</u> in a packet (DSCP) or frame. For example, an <i> IP
packet</i> has the ToS (Type of Service) field that we can use to mark
the packet.
<i>Ethernet frames</i> don’t have such field but we do have something for
trunks. The tag that is added by 802.1Q has a priority field.
* Classification: At layer 4 to 7 using NBAR (Network-Based Application Recognition)
* Traffic Marking: At layers 2 and 3; as close to the source as possible
  * Ethernet (802.1Q, 802.1p): Layer 2, Field: Class of Service (CoS), 3 bits
  * 802.11 (Wi-Fi): Layer 2, Field: TID, 3 bits
  * MPLS: Layer 2, Field: Experimental (EXP): 3 bits
  * IPv4 and IPv6: Layer 3, Field: ToS, IP Precedence, 3 bits
  * IPv4 and IPv6: Layer 3, Field: DSCP, 6 bits 

## Congestion avoidance
### Buffer Management
* It is best to drop packets as soon as there is congestion
* If there is congestion in a particular port, it is difficult to know whose fault it is (Or who is the source that has
caused the congestion)
* Random Early Detection (RED): This is a proactive approach in which the router discards one or more packets before the buffer becomes completely full
 
The congestion avoidance tool can randomly drop packets from output queue, or we can
configure it to give certain packets a different treatment based on their marking. Main tool is weighted random early detection or WRED
* **Policing and selective dropping:**
  * Policing: do so by discarding traffic. (Policing is often used by ISPs who must limit the bitrate of their customers to Committed Information Rate (CIR).)
      * Drops exceeding traffic: which is bad. that is why we use shaping on our enterprise.
      Policing drops exceeding traffic while shaping buffers exceeding traffic
      * Use when you are trying to limit a specific traffic type from stealing all of the bandwidth
      * Inbound or outbound
      * For example we can use policing for Windows update to be dropped if it is stealing our bandwidth.
* Mechanisms

    ![image](https://user-images.githubusercontent.com/31813625/40928919-f4ec0f54-67f0-11e8-814c-17346507a6d1.png)

## Congestion management
  
* **Queuing and scheduling:**
provides congestion management. Packets are queued during congestion to prevent drops (tail-drops).
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
* **Shaping:**
  * Shaping: will hold packets in a queue, adding delay, to avoid bursty traffic.
    * Buffers the exceeding traffic
    * Use when you are trying to match the bandwidth on a WAN link. For example you 
      have a 100Mi interface but your CIR is 20Mi. Shaping makes sure that the traffic rate
      you are sending to the provider matches what they are limiting you to.
    * Only outbound
    * *Simple algorithms*; implemented in Data Plane.