# Data link layer
### Purpose
  * Ensure reliable, efficient communication between neighbor nodes
* Four specific functions:
  * Provide services to network layer
  * Framing
  * Error handling
  * Flow control

### Service Provided to the "Network Layer"

#### 1. Unacknowledged connectionless
* Just send frames towards the destination
* No connection is established, No connection released
* No acknowledge of received frames
* No attempt to recover lost frames
* Appropriate in case of:
  * Real-time traffic: speech, video: Short delay is more important than 100% reliability
  * Low error channels: leave error correction to higher layers
  * Most LAN’s are using this service class
* Example: Ethernet

#### 2. Acknowledged connectionless
* Each frame is acknowledged, but no connection is established
* Why don't we acknowledge in layer 3. Why layer 2 does the acknowledgment?
  * Because let's say Layer 3 gives 30,000 Bytes to Layer 2, Layer 2 then sends these packets for example 1500 Bytes - 1500 Bytes as frames.
  If ack was in layer 3, then if one bit failed we had to retransmit those 30,000 bytes. But when the ack is in layer 2, we only need to
  send those 1500-byte size frame. So less delay and load we are having on the channel
* Acknowledgment is a service that can be performed by the transport layer as well
* Data link layer provides this service to avoid long delays that could be caused if frames are not acknowledged
* Specially important over un-reliable channels such as wireless
* Example: 802.11 (WiFi)

#### 3. Acknowledged Connection Oriented
* Establishes a connection before sending data
* Each frame is numbered
* Data link layer guarantees the delivery of a SINGLE copy of EVERY frame (With acknowledged connectionless, it is possible to receive multiple copies of a frame due to a lost ACK)
* Releases the connection at the end of conversation (To release software and hardware resources tied up to the connection)
* Provides a reliable bit stream to NL

### Framing
Since the physical layer merely accepts and transmits a stream of bits
without any regard to meaning or structure, it is upto the data link layer
to create and recognize frame boundaries. This can be accomplished by attaching
special bit patterns to the beginning and end of the frame. If these bit
patterns can accidentally occur in data, special care must be taken to make
sure these patterns are not incorrectly interpreted as frame delimiters.

### Error Detection and Correction
The most common error control method is to compute and append some form of a
checksum to each outgoing frame at the sender's data link layer and to recompute
the checksum and verify it with the received checksum at the receiver's side.
If both of them match, then the frame is correctly received; else it is erroneous.

The checksums may be of two types:
* **Error detecting:** Receiver can only detect the error in the frame and inform the sender about it
* **Error detecting and correcting:** The receiver can not only detect the error but also correct it

But each Ethernet frame carries a Cyclic Redundancy Check (CRC)-32 checksum.

### Flow Control
Consider a situation in which the sender transmits frames faster than the
receiver can accept them. If the sender keeps pumping out frames at high rate,
at some point the receiver will be completely swamped and will start losing
some frames. This problem may be solved by introducing flow control. Most flow
control protocols contain a feedback mechanism to inform the sender when it should transmit the next frame.

## 802.3 LAN Frame

![2000px-ethernet_802 1q_insert svg](https://user-images.githubusercontent.com/31813625/35241232-d4323944-ff83-11e7-8e0c-701a4b8cfde6.png)

* **Preamble (7 Bytes):** this is a 7-byte pattern of ones and zeroes and is used for synchronization
* **SFD (1 Byte):** the "start frame delimiter" marks the end of the preamble and tells the receiver that the next fields will be the actual Ethernet frame, starting with the destination field
* **Destination (2 or 6 Bytes):** this is the destination MAC address of the receiver
* **Source (2 or 6 Bytes):** the source MAC address of the device that sent the frame
* **802.1Q (4 Bytes):** adds a 32-bit field between the source MAC address and the EtherType fields of the original frame 
* **Type/Size (2 Bytes):** this tells us what is carried inside the Ethernet frame. An IPv4 packet, IPv6 packet or something else
* **Data (46-1500 Bytes):** this carries the actual data that we are trying to transmit, for example an IPv4 packet. If the length of the field is less than 46 bytes, then padding data is added to bring its length up to the required minimum of 46 bytes
* **Frame Check Sequence FCS (4 Bytes):** It contains a 32 bit Cyclic Redundancy Check (CRC). The frame check sequence helps the receiver to figure out if the frame is correct or corrupt