# Quality of Service (QoS):
Internet is best effort of FIFO (First In First Out). This is not QoS.

QoS controls help you better manage available bandwidth. QoS is about using tools to change how the router or switch
deals with different packets. For example, we can configure the router so that voice traffic is prioritized before data traffic.

## When QoS is needed?

Network congestion causes delay.
* Examples of typical congestion points

    ![qos](https://user-images.githubusercontent.com/31813625/40891958-8cc737e8-675d-11e8-985a-4c4cccb3ffb6.png)

## Characteristics of network traffic
There are four characteristics of network traffic that we must deal with:
#### 1. Bandwidth
the speed of the link. bits per second (bps)
#### 2. Congestion
An interface experiences congestion when it is presented with more traffic than it can
handle. Network congestion points are strong candidates for QoS mechanisms. 
* What is the difference between congestion control and flow control?
  * Congestion control
    * Global issue (network)
    * Indirect feedback
    * Sometimes slowdown messages are used
  * Flow control
    * Local issue (line, point-to-point connection )
    * Direct feedback from neighbor
* Time Scale of applying congestion control techniques

    ![image](https://user-images.githubusercontent.com/31813625/40932368-203be21e-67fc-11e8-83c7-f08d49cf078c.png)
  
  * Network provisioning: network design
  * Traffic-aware routing: avoid congested paths/routes
  * Admission control: if network is congested, don't accept new traffic
  * Traffic throttling: bring down the rate of the network device which is sending packets in a higher rates
  * Load shedding: discard packets

#### 3. Delay (or latency)
we have one-way (time from source to destination) and two-way delay (from source to destination and back)
  * Processing delay: the time for a device to perform the tasks
  * Queuing delay: the time that a packet is waiting in the memory until resources become available to transmit it
  * Serialization delay: the time that takes to send all bits of a frame from NIC to the PHY for transmission
  * Propagation delay: the variable amount of time it takes for bits to cross the PHY. We cannot change this type of delay.
  * Code delay: The fixed amount of time time it takes to compress data at the source before transmitting
  to the first internetworking device

#### 3. Jitter
variation of one-way delay. The receiver of packets (for voice experience)
must deal with jitter, making sure the packets have a steady delay. 

![image](https://user-images.githubusercontent.com/31813625/40934269-42eeab88-6802-11e8-93db-0154cbd1dee0.png)

#### 4. Loss
Without any QoS mechanisms in place, packets are processed in the order in which they are received.
When congestion occurs, network devices such as routers and switches can drop packets. This means that
time-sensitive packets, such as real-time video and voice, will be dropped with the same frequency as
data that is not time-sensitive, such as email and web browsing.
Usually as a percentage of lost packets sent. When there is a congestion,
packets will be queued but once the the queue is ful, packets will be dropped. With QoS
we can decide which packets drop when this happens. In a properly designed network, packet loss should be near zero.

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

  | Voice | Video |
  | --- | --- | 
  | Smooth | Bursty |
  | Benign | Greedy |
  | Drop sensitive | Drop sensitive |
  | Delay sensitive | Delay sensitive |
  | UDP priority | UDP priority |
  | Latency <= 150 ms | Latency 200-400 ms |
  | Jitter <= 30 ms | Jitter <= 30 - 50 ms |
  | Loss <= 1% | Loss <= 0.1% - 1$ |
  | Bandwidth >30 | >384Kbps |

* Traffic Engineering table

| Application | Reliability | Delay | Jitter | Bandwidth |
| --- | --- | --- | --- | --- |
| E-mail | High | Low | Low | Low |
| File transfer | High | Low | Low | Medium |
| Web access | High | Medium | Low | Medium |
| Remote login | High | Medium | Medium | Low |
| Audio on demand | Low | Low | High | Medium |
| Video on demand | Low | Low | High | High |
| Telephony | Low | High | High | Low |
| Videoconferencing | Low | High | High | High |

## QoS Algorithms

### First-In, First-Out (FIFO) Queuing
the absence of QoS; also known as first-come, first-served (FCFS) queuing.
Packets are sent out an interface in the order in which they arrive, as shown in the figure.
    
  ![fifo](https://user-images.githubusercontent.com/31813625/40922218-aab92a54-67df-11e8-836f-3e0023d1b671.png)

All interfaces except serial interfaces at E1 (2.048 Mbps) and below use FIFO by default. (Serial interfaces at E1 and below use WFQ by default.)
FIFO, which is the fastest method of queuing, is effective for large links that have little delay and minimal congestion.
If your link has very little congestion, FIFO queuing may be the only queuing you need to use

### Weighted Fair Queuing (WFQ)

WFQ applies priority, or weights, to identified traffic and classifies it into conversations or flows, as shown in the figure.

  ![priority](https://user-images.githubusercontent.com/31813625/40922219-aac6bdcc-67df-11e8-82e3-1a737a827e89.png)

WFQ then determines how much bandwidth each flow is allowed relative to other flows.
The flow-based algorithm used by WFQ simultaneously **schedules** interactive traffic to the front of a queue to reduce
response time. It then fairly shares the remaining bandwidth among high-bandwidth flows. WFQ allows you to give
low-volume, interactive traffic, such as Telnet sessions and voice, priority over high-volume traffic, such as FTP
sessions.

WFQ classifies traffic into different flows based on packet header addressing, including such characteristics as source
and destination IP addresses, MAC addresses, port numbers, protocol, and Type of Service (ToS) value.
The ToS value in the IP header can be used to classify traffic.

**Limitations**

WFQ is not supported with tunneling and encryption because these features modify the packet content information required
by WFQ for classification.

Although WFQ automatically adapts to changing network traffic conditions, it does not offer the degree of precision
control over bandwidth allocation that CBWFQ offers.

### Class-Based Weighted Fair Queuing (CBWFQ): newer

CBWFQ extends the standard WFQ functionality to provide support for user-defined traffic classes.
For CBWFQ, you define traffic classes based on match criteria including protocols, access control lists (ACLs),
and input interfaces. Packets satisfying the match criteria for a class constitute the traffic for that class.
A FIFO queue is reserved for each class, and traffic belonging to a class is directed to the queue for that class,
as shown in the figure.

  ![cbwfq](https://user-images.githubusercontent.com/31813625/40922220-aad60e1c-67df-11e8-98b9-0814d08acb1a.png)

After a queue has reached its configured queue limit, adding more packets to the class causes tail drop or packet drop
to take effect, depending on how class policy is configured. Tail drop means a router simply discards any packet that
arrives at the tail end of a queue that has completely used up its packet-holding resources. This is the default
queuing response to congestion. Tail drop treats all traffic equally and does not differentiate between classes of service.

### Low Latency Queuing (LLQ):
Can be used with CBWFQ to prioritize the voice packets above all else.

The LLQ feature brings strict priority queuing (PQ) to CBWFQ. Strict PQ allows delay-sensitive data such as voice to be
sent before packets in other queues. LLQ provides strict priority queuing for CBWFQ, reducing jitter in voice
conversations, as shown in the figure.

![llq](https://user-images.githubusercontent.com/31813625/40924817-31611b74-67e6-11e8-8d86-fc0fea85d45b.png)

Although it is possible to enqueue various types of real-time traffic to the strict priority queue,
Cisco recommends that only voice traffic be directed to the priority queue.

## How can QoS be implemented in a network?
The three models for implementing QoS are:
* Best-effort model
* Integrated services (IntServ)
* Differentiated services (DiffServ)

<table>
  <tr>
    <th>Model</th>
    <th>Description</th> 
  </tr>
  <tr>
    <td>Best-effort model</td>
    <td><ul>
      <li>Not really an implementation as QoS is not explicitly configured. </li>
      <li>Use when QoS is not required</li>
      <li>This approach is still predominant on the Internet today and remains appropriate for most purposes</li>
      <li>The model is the most scalable</li>
      <li>Scalibility is only limitted by bandwidth</li>
      <li>No packets have preferential treatment</li>
      <li>The best-effort model is similar in concept to sending a letter using standard postal mail. Your letter is treated exactly the same as every other letter.</li>
      </ul>
    </td> 
  </tr>
  <tr>
    <td>Integrated services (IntServ)</td>
    <td><ul>
      <li>Provides very high QoS to IP packets with guarantied delivery.</li>
      <li>IntServ uses a connection-oriented approach inherited from telephony network design. Each individual communication must explicitly specify its traffic descriptor and requested resources to the network</li>
      <li>It defines a signaling process for applications to signal to the network that they require special QoS for a period and that bandwidth should be reserved.</li>
      <li>In the IntServ model, the application requests a specific kind of service from the network before sending data. The application informs the network of its traffic profile and requests a particular kind of service that can encompass its bandwidth and delay requirements.</li>
      <li>IntServ uses the <b>Resource Reservation Protocol (RSVP)</b> to signal the QoS needs of an application’s traffic along devices in the end-to-end path through the network. </li>
      <li>IntServ can severely limit tghe scalability of a network.</li>
      </ul>Critical resources<ul>
      <li>Bandwidth</li>
      <li>Buffer space</li>
      <li>CPU processing cycle</li>
      </ul></td>
  </tr>
  <tr>
  <td>Differentiated services (DiffServ)</td>
  <td><ul>
    <li>Provides high scalability and flexibility in implementing QoS.</li>
    <li>The DiffServ model is similar in concept to sending a package using a delivery service. You request (and pay for) a level of service when you send a package. Throughout the package network, the level of service you paid for is recognized and your package is given either preferential or normal service, depending on what you requested.</li>
    <li>DiffServ is not an end-to-end QoS strategy because it cannot enforce end-to-end guarantees.</li>
    <li>Network devices recognize traffic classes and provide different levels of QoS to different traffic classes.</li>
    </ul></td> 
  </tr>
</table>

## QoS Implementation Tools

QoS Sequence:

![image](https://user-images.githubusercontent.com/31813625/40927338-7cb590b8-67ec-11e8-8a2e-93bb6ec4fb4d.png)


### Classification and marking tools
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

### Congestion avoidance
#### Buffer Management
* It is best to drop packets as soon as there is congestion
* If there is congestion in a particular port, it is difficult to know whose fault it is (Or who is the source that has
caused the congestion)
* Random Early Detection (RED): This is a proactive approach in which the router discards one or more packets before the buffer becomes completely full
 
The congestion avoidance tool can randomly drop packets from output queue, or we can
configure it to give certain packets a different treatment based on their marking. Main tool is weighted random early detection or WRED
* **Policing and selective dropping:**
  * Policing: do so by discarding traffic. (Policing is often used by ISPs who must limit the bitrate of their customers to Committed Information Rate (CIR).)
      * Drops exceeding traffic
      * Use when you are trying to limit a specific traffic type from stealing all of the bandwidth
      * Inbound or outbound
      * For example we can use policing for Windows update to be dropped if it is stealing our bandwidth.
* Mechanisms

    ![image](https://user-images.githubusercontent.com/31813625/40928919-f4ec0f54-67f0-11e8-814c-17346507a6d1.png)

### Congestion management
  
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

## Type of Service (TOS)
IP packets have a field called the Type of Service field (TOS byte)

### IPv4 and IPv6 Packet Header

![image](https://user-images.githubusercontent.com/31813625/40928868-d1b379e6-67f0-11e8-8bd0-22f24801eda6.png)

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

![image](https://user-images.githubusercontent.com/31813625/40928285-ed84d860-67ee-11e8-819e-7579eac2e3da.png)

It is found in the header of 802.1Q (layer 2). It’s used for Quality of Service on trunk links.
(Priority field on 802.1Q tag).
  * Values 0-7: higher value, higher priority.
  
  | Value | Description |
  | --- | --- |
  | 7 | Reserved |
  | 6 | Reserved |
  | 5 | Voice bearer (voice traffic) |
  | 4 | Videoconferencing |
  | 3 | Call Signaling |
  | 2 | High-Priority Data |
  | 1 | Medium-Priority Data |
  | 0 | Best-Effort Data |

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