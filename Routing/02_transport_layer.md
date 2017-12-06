# Transport Layer
* Tasks:
  * Provide reliable and cost effective end-to-end communication service to the application layer
  * Independent of used network (shielding)
  * Boundary between network and applications
* TPDU (transport protocol data unit) AKA "segment" denotes message sent from entity to entity
* Can be both connection oriented and connectionless service

## Transport Services
* *Transport Services* vs *Data Link* Services
  * Similarities:
    * Both provide point-to-point connection
    * Both have to deal with error control, sequencing, flow control, retransmission, etc
  * Differences:
    * Transport connection is indirect
    * Network has memory: packets may be stored and and arrive with varying delays and out of order
      * In Data link, when we send first and send frame to PHY, we receive them in order. But in transport layer, when sending segments
    maybe we receive the second segment prior to th first segment
    * Many connections have to be managed (instead of a fixed number of links)
* Primitives for a simple transport service:
  * LISTEN: block until some process connects
  * CONNECT
  * SEND
  * RECEIVE: block until data TPDU arrives
  * DISCONNECT
* TCP addressing:
  * IP protocol number, TCP/UDP port number

## Transmission Control Protocol (TCP)
* TCP Provides a logical full duplex connection between two application layer
processes across an unreliable datagram network (IP network)
* TCP Provides flow control using Selective Repeat
* TCP Can support multiple connections at the same time
* Each direction of the connection can be terminated independently
* TCP Service is:
  * Connection oriented
  * Reliable
  * In-sequence
  * Byte-stream oriented
* TCP Service Model
  * TCP connection is :
    * Full-duplex, point-to-point (No support for multicast or broadcast)
    * Byte stream : message boundaries are not preserved (Application layer should parse and detect messages)
* Both sender and receiver have to create sockets
  * socket # = IP address + Port# (16-bit, = TSAP)
* Connection is identified by: (socket 1, socket 2)
* Port# < 1024 : well-known ports
* FTP: 21, Telnet:23, SMTP: 25, HTTP: 80
* TCP may buffer at both sides; consequently transmission may be delayed
* TCP should handle lost, out of order, and delayed segments with non-aligned boundaries
* Every byte has 32-bits *sequence number*
  * On 10 Mbps it takes about an hour to wrap around
  * Separate 32-bit sequence numbers are used for acknowledgement and for the window mechanism
* Data exchange in segments
  * Segment contains 20-byte header, options, + data
  * Variable payload size (the size is decided by the TCP software and is usually 1500 bytes)
  * Must fit into MTU: maximum transfer unit size of a network
* **Sliding window** protocol with timeout
  * Receiver sends back ack# equal to next expected segment#
  * Receiver uses piggybacking. Meaning that I send you something, You want to send me something. Okay you can give me the ack of the segment
  I had sent you in the header of the TCP you wan to send to me
  * Sender retransmits if timeout occurs

### TCP Congestion Control
* TCP controls the data flow between two endpoints of a connection
* **Selective repeat sliding window** is used for flow control
* Receiver specifies its reception capacity called "Advertised Window" in the header of the
messages that it sends back to transmitter. This handles the end-to-end flow control
* Network congestion can also be controlled by slowing down the flow
of data that TCP generates into the network. This requires feedback from the network behavior
* The maximum amount of bytes that a TCP sender can transmit
without congesting the network is called "Congestion Window"
* TCP works by adapting these two windows to control the traffic generation process
* Sender can transmit up to the minimum of "Advertised Window" and
"Congestion Window". This is called "Current Window"
* Timer management:
  * Round trip delay of each segment (Transmission to ack) is measured.
  * Appropriate time is chosen based on history and variation of this delay
  * If timeout happens, it assumes that congestion happened then it reduces the transmission rate
* One method is *slow start mode:*
  * In slow start mode, the congestion window size increases by one Maximum
Segment Size (MSS) per received ack which results in
its doubling once every RTT.
(MSS indicates the maximum number
of data bytes any particular layer-2 technology allows per
packet. TCP/IP headers are later added to the packet (normally
40 bytes) and the resulting total is called the Maximum
Transmission Unit (MTU))
* Another method is *congestion avoidance:*
  * In the congestion avoidance mode the window size increases by one MSS every RTT.
When a packet is lost, the window size is halved. TCP thus
uses an additive increase, multiplicative decrease algorithm
for congestion control

###
```

   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```
* **Source Port:**  16 bits
  * The source port number
* **Destination Port:**  16 bits
  * The destination port number
* **Sequence Number:**  32 bits
  * The sequence number of the first data octet in this segment (except
    when SYN is present). If SYN is present the sequence number is the
    initial sequence number (ISN) and the first data octet is ISN+1
* **Acknowledgment Number:**  32 bits
  * If the ACK control bit is set, this tells the sender I received until
  byte n you had sent to me
* **Data Offset:**  4 bits
  * The length of the header
* **Reserved:**  6 bits
  * Reserved for future use.  Must be zero.
* **Control Bits:**  6 bits (from left to right):
  * **URG:**  The URG flag is used to inform a receiving station that certain
  data within a segment is urgent and should be prioritized. If the URG
  flag is set, the receiving station evaluates the **urgent pointer**, a 16-bit field
  in the TCP header. This pointer indicates how much of the data in the segment,
  counting from the first byte, is urgent.
  The URG flag isn't employed much by modern protocols, but we can see
  an example of it in the Telnet packet capture
  * **ACK:**  Acknowledges received data
  * **PSH:**  The socket that TCP makes available at the session level can be
  written to by the application with the option of "pushing" data out immediately,
  rather than waiting for additional data to enter the buffer
  * **RST:**  Aborts a connection in response to an error
  * **SYN:**  Initiates 3-way handshake and Sequence Number initiation
  * **FIN:**  No more data from sender. Closes a connection
* **Window:**  16 bits
  * The number of data octets beginning with the one indicated in the
    acknowledgment field which the sender of this segment is willing to
    accept.
* **Checksum:**  16 bits
  * Used for error detection

## User Datagram Protocol
* UDP in an unreliable, connectionless transport protocol
* UDP Services:
  * Routes the received packet to the desired application on the host
(Destination Port)
  * Checks the integrity of the datagram. This is optional! (UDP Checksum)
* If a host does not wish to calculate the checksum, it sets it to all 0's.
* Applications that use UDP: Domain Name Services (DNS), Simple
Network Management Protocol (SNMP), Real Time Protocol (RTP)
* UDP checksum calculation: Similar to TCP (pad to 16, psuedoheader
for IP verification)

### UDP Header Format
```
      0      7 8     15 16    23 24    31
     +--------+--------+--------+--------+
     |     Source      |   Destination   |
     |      Port       |      Port       |
     +--------+--------+--------+--------+
     |                 |                 |
     |     Length      |    Checksum     |
     +--------+--------+--------+--------+
     |
     |          data octets ...
     +---------------- ...
```

## Real Time Transport Protocol (RTP)
* A generic protocol for real time applications such as voice and video
  * Uses UDP and acts as an interface between user application and transport
protocol (Mostly UDP)
* Header specifies the profile and encoding format of the payload (single
audio stream, mp3)
* Packets are numbered to allow detection of missing packets
* Time stamping is used to allow synchronization and jitter compensation
* No flow control, no error control, no acknowledgement, and no repeat-requests
* Services:
  * Payload type identification
  * Sequence numbering
  * Time stamping
  * Delivery monitoring

### RTP Fixed Header Fields
```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |V=2|P|X|  CC   |M|     PT      |       sequence number         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                           timestamp                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           synchronization source (SSRC) identifier            |
   +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
   |            contributing source (CSRC) identifiers             |
   |                             ....                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
* V = version (V): 2 bits
* P = padding (P):
  * 1 bit - the packet has been padded to a multiple of 4-bytes. The last padding
byte tells how many bytes were padded
* extension (X): 1 bit
  * If the extension bit is set, the fixed header MUST be followed by
      exactly one header extension
* CSRC count (CC): 4 bits
  * he CSRC count contains the number of CSRC identifiers that follow
      the fixed header
* marker (M): 1 bit - used by the application
* payload type (PT): 7 bits
  * identifies the format of the RTP payload and determines its interpretation by the application.
  * A receiver MUST ignore packets with payload types that it does not
      understand
* sequence number: 16 bits
  * The sequence number increments by one for each RTP data packet
      sent, and may be used by the receiver to detect packet loss and to
      restore packet sequence
* timestamp: 32 bits
  * Appropriate time alignment of the samples in the receiver for smooth playback
and jitter reduction
  * Synchronization between multiple streams such as video and audio in a
video conference
* SSRC: 32 bits: Tells which stream this packet belongs to
* CSRC list: 0 to 15 items, 32 bits each

### Real Time Transport Control Protocol
* RTP is used in conjunction with the RTP Control Protocol (RTCP).
* RTP carries the media streams (e.g., audio and video) and RTCP is
used to monitor transmission statistics and quality of service (QoS)
and aids synchronization of multiple streams.
* When both protocols are used in conjunction, RTP is originated and
received on even port numbers and the associated RTCP
communication uses the next higher odd port number.
* RTCP feedback can let the source know about delay, jitter,
bandwidth, congestion, etc.
* This data can be used by the source to adjust encoding or transport
properties of the stream
* Different streams can use different clocks with different granularities
and different drift rates.