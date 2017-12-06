# Transport Layer
* Tasks:
  * Provide reliable and cost effective end-to-end communication service to the application layer
  * Independent of used network (shielding)
  * Boundary between network and applications
* TPDU (transport protocol data unit) AKA "segment" denotes message sent from entity to entity
* Can be both connection oriented and connectionless service

## Transport Services
* *Transport Services* vs *Data Link Services*
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

### Transmission Control Protocol (TCP)
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