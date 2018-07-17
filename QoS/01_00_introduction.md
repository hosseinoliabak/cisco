# Quality of Service (QoS):

* IP has been originally designed to provide "best effort" service
* Internet is best effort of FIFO (First In First Out). This is not QoS
* Over provisioning of IP networks is an easy option to start with
* In many cases, congestion that causes degraded quality of service is a local issue in the network of an ISP
* Bandwidth cost is steadily decreasing, thereby many ISPs use this method to address the problem
* This method cannot ensure the necessary QoS; for example jitter cannot be controlled by over provisioning
* QoS are tools which tunes bandwidth, delay, jitter, and loss for every your interested traffic
* Some critical applications are moving to IP
  * Voice over IP
  * Grid computing
* QoS is not limited to IP network and packet level traffic control. The research in this area has extended into 
wireless and optical domain as well
  * 802.11e: Service differentiation in WLAN
  * Automatic configuration and management in next generation optical networks (ASON)
  
## When QoS is needed?

Network congestion causes delay.
* Examples of typical congestion points

    ![qos](https://user-images.githubusercontent.com/31813625/40891958-8cc737e8-675d-11e8-985a-4c4cccb3ffb6.png)

## Characteristics of network traffic
There are four characteristics of network traffic that we must deal with:

#### 1. Bandwidth
The speed of the link. bits per second (bps). This is important for example for FTP traffic

Tools that affect Bandwidth:
* Compression: Compresses either payload or header, reducing overall number of bits 
required to transmit the data
* CAC (call admission control): Reduces overall load introduced into the network by
rejecting new voice and video calls 
* Queuing (congestion management): Can be used reserve minimum amounts of bandwidth
for particular types of packets


#### 2. Delay (or latency)
we have one-way (time from source to destination) and
two-way delay (from source to destination and back)
  * Serialization delay (fixed)(negligible on T3 and faster links): the time that takes to send all bits of a frame from NIC to the
  PHY for transmission.
    * serialization delay (milliseconds) = number of bits sent ÷ link speed (kbps)
  * Propagation delay (fixed)(not significant): the variable amount of time it takes for bits to cross the PHY.
  We cannot change this type of delay.
    * Propagation delay (milliseconds) = Length of the link (meters) ÷ (2.1 × 10<sup>8</sup> meters/seconds)
  * Forwarding/Processing delay (variable)(not really significant): the time for a device to perform the tasks
  * Queuing delay (variable)(could be significant): the time that a packet is waiting in the memory until resources become available to transmit it
  For voice should be almost 0
  * Packetization delay (fixed): The IP phone or voice gateway must collect 20 ms of voice before it can 
  put 20 ms worth of voice payload into a packet
  * Codec delay (fixed): The fixed amount of time time it takes to compress data at the source before transmitting
  to the first internetworking device; usually between 2.5 ms to 10 ms
    * This delay overlaps with packetization delay
  * De-jitter buffer delay (variable): At receiving side. receiving side buffers for example 2 packets (40 ms), then
  when the 3rd packet arrives, it delivers 1st packet. Why? to maintain the jitter as lowest as possible. For
  **interactive videos**, this delay can be up to 70 ms. De-jitter buffer for **streaming videos** can run into the tens of seconds!
  * Shaping delay (variable): Sending packets more slowly, but not having them be dropped. (traffic shaping)
  * Network delay (variable): In some cases, the provider will include delay limits in the contracted SLA

Tools that affect delay:
* Queuing: enables you to order packets so that delay-sensitive packets leave their queues more quickly
than delay-insensitive packets
* Link fragmentation and interleaving: Because routers do not preempt a packet that is currently being
transmitted, LFI breaks larger packets into smaller fragments before sending them. Smaller delay-sensitive
packets can be sent after a single smaller fragment, instead of having to wait for larger original packet
to be serialized
* Compression
* Traffic shaping: Artificially increases delay to reduce drops inside a Frame Relay or ATM network

#### 3. Jitter
variation of one-way delay. The receiver of packets (for voice experience)
must deal with jitter, making sure the packets have a steady delay. 

![image](https://user-images.githubusercontent.com/31813625/40934269-42eeab88-6802-11e8-93db-0154cbd1dee0.png)

Tools that affect jitter: (the same as delay)
* Queuing
* Link fragmentation and interleaving
* Compression
* Traffic shaping

#### 4. Loss
Without any QoS mechanisms in place, packets are processed in the order in which they are received.
When congestion occurs, network devices such as routers and switches can drop packets. This means that
time-sensitive packets, such as real-time video and voice, will be dropped with the same frequency as
data that is not time-sensitive, such as email and web browsing.
Usually as a percentage of lost packets sent. When there is a congestion,
packets will be queued but once the the queue is ful, packets will be dropped. With QoS
we can decide which packets drop when this happens. In a properly designed network, packet loss should be near zero.

It is important for example for UDP traffics

Tools that affect loss:
* Queuing: longer queue increases delay, but avoids loss
* RED (random early detection or congestion avoidance): drops packet randomly as queues approach
the point of being full, slowing some TCP connections

## Congestion
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
  | Latency <= 150 ms (ITU); <= 200 ms (Cisco) | Latency 200-400 ms |
  | Jitter <= 30 ms | Jitter <= 30 - 50 ms |
  | Loss <= 1% | Loss <= 0.1% - 1$ |
  | Bandwidth >30 | >384Kbps |

* Traffic Engineering table

| Application | Bandwidth | Delay | Jitter | Loss |
| --- | --- | --- | --- | --- |
| Voice payload | Low | Low | Low | Low |
| Video payload interactive (Videoconferencing) | High | Low | Low | Low |
| Video payload 1-way (streaming) | High | High | High | Low |
| Interactive data (Remote login, SSH) | Medium | Medium | Medium | Medium |
| Interactive data, not critical | Medium | High | High | Medium |
| Not interactive data, mission critical | High | High | High | Medium |
| Not interactive data, not critical | High | High | High | High |

## Planning and Implementing QoS Policies
<ol>
<li>Identify traffic and its requirement<ul>
  <li>perform a network audit by using trace analysis tools, management tools, Network-Based Application Recognition (NBAR),
  or any other tool to identify protocols and traffic volume</li>
  <li>perform a business audit to determine the importance of the discovered traffic types to the business</li>
</ul>
<li>Divide traffic into classes<ul>
  <li>The number of service classes will vary from site to site. However, for voice and video, you will 
  likely end up with 3 classes<ul>
    <li>One for voice payload</li>
    <li>One for video payload</li>
    <li>One for voice and video signaling traffic</li></ul></li>
  <li>For data, Cisco tends to highlight the following general classes in the QoS course<ul>
    <li>Mission critical</li>
    <li>Transactional (interactive)</li>
    <li>Best-effort</li>
    <li>Scavenger</li></ul></li>
</ul>    
<li>Define QoS policies for each class<ul>
  <li>Bandwidth limiting</li>
  <li>Guarantee bandwidth</li>
  <li>Guarantee delay and jitter</li>
  <li>Not packet loss</li></ul></li>
</ol>

## Key Element of QoS Management
* Per packet operations
  * Shaping
  * Policing
  * Classification
  * Scheduling
  * Queue Management
* Per Flow Operations
  * Admission Control
  * Resource Management
  * Signaling
  * Routing
  * Resource Management
* Congestion Control
* QoS Pricing