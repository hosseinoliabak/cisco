# IP routing Overview
### What is IP Routing
IP Routing is the process of delivering IP Packets from one device to another, across an IP network, using routers.

### How the routers learn about these routes?
* Static Routing
* Using a Dynamic Routing Protocol
  * IGP: Allow Routers to share routing information with each other
  * EGP: The basic concept of all EGP protocols is to share information
about reachable IP networks but hide the network topology

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
* **Version:** Always 4
* **IP Header Length:** Tells us the number of 32 -bit words forming the header,
the minimum length of an IP header is 20 bytes so with 32-bit increments, we would see value of 5 in this field.
* **Type of Service (AKA TOS Byte):** Used for quality of service desired
  *  The original idea behind the TOS byte was that we could specify a priority and request a route for high throughput, low delay and high reliable service
  The first 3 bits are used to define a precedence. The higher the value,
  the more important the IP packet is, in case of congestion the router would drop
  the low priority packets first.
    * Precedence
      * 111 - Network Control
      * 110 - Internetwork Control
      * 101 - CRITIC/ECP
      * 100 - Flash Override
      * 011 - Flash
      * 010 - Immediate
      * 001 - Priority
      * 000 - Routine
  The next 5 bits are type of service bits which are used to
  assign what kind of delay, throughput, and reliability we want
    * Type of Service:
      * Bit 3 (never been used):	0 = normal delay;	1 = low delay
      * Bit 4 (never been used):	0 = normal throughput;	1 = high throughput
      * Bit 5 (never been used):	0 = normal reliability;	1 = high reliability
      * Bit 6-7 (never been used):	Reserved for future use
  * The TOS byte has been defined back in 1981 in RFC 791 but the way we use it has changed throughout the years.
   there is a lot of terminology and some of is not used anymore nowadays. in 1992 RFC 1349 was created that changes\
   the definition of the TOS byte. The first 3 precedence bits remain unchanged but the type of service bits have changed.
  ```
          0     1     2     3     4     5     6     7
       +-----+-----+-----+-----+-----+-----+-----+-----+
       |                 |                       |     |
       |   PRECEDENCE    |          TOS          | MBZ |
       |                 |                       |     |
       +-----+-----+-----+-----+-----+-----+-----+-----+
  ```
    * Type of service:
      *1000   --   minimize delay
      * 0100   --   maximize throughput
      * 0010   --   maximize reliability
      * 0001   --   minimize monetary cost
      * 0000   --   normal service
    * we now only use 4 bits to assign the type of service and the final bit is called MBZ
    (Must Be Zero).  Routers will ignore this bit.
    * But the type of service bits have never been really used
    * So what do we actually use nowadays? It is discussed in "Quality of Service" course
* **Total Length:** Total Length is the entire length of the IP packet, including header and data
  * 20 Bytes - 65,535 Bytes
* **Identification:** If the IP packet is fragmented then each fragmented packet will use the same 16 bit identification number to identify to which IP packet they belong to
* **IP Flags:** These 3 bits are used for fragmentation
  * Bit 0: reserved, must be zero
  * Bit 1: (DF) 0 = May Fragment,  1 = Don't Fragment.
  * Bit 2: (MF) 0 = Last Fragment, 1 = More Fragments.
* **Fragment Offset:** This field indicates where in the datagram this fragment belongs
* **Time to Live:** This field indicates the maximum time the datagram is allowed to remain in the internet system.
Every time an IP packet passes through a router, the time to live field is decremented by 1.
Once it hits 0 the router will drop the packet and sends an ICMP time exceeded message to the sender
* **Protocol:** This field indicates the next level protocol used in the data
portion of the internet datagram.
  * Here is the list of IP protocol numbers: https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml
  * For example TCP has value 6 and UDP has value 17
* **Header Checksum:** A checksum on the header only.  Since some header fields change
    (e.g., time to live), this is recomputed and verified at each point
    that the internet header is processed
* **Source Address:**
* **Destination Address:**
* **IP Option:** This field is not used often

## Internet Protocol, Version 6 (IPv6)
* ARP has been replaced by ICMPv6 Neighbor Discovery Protocol.
* Multicast instead of broadcast
* Minimum MTU of 1280 Bytes


### IPv6 Header Format:
* 40-Byte Main/Regular IPv6 Header
```
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Traffic Class |           Flow Label                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Payload Length        |  Next Header  |   Hop Limit   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                         Source Address                        +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                      Destination Address                      +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
* **Version:** 4-bit Internet Protocol version number = 6.
* **Traffic Class:** 8-bit traffic class field. The most significant 6 bits are
used for Type of Service to let the Router Known what services should be
provided to this packet. The least significant 2 bits are used for Explicit Congestion Notification (ECN).
* **Flow Label:** 20-bit Flow Label field helps avoid re-ordering of data packets.
It is designed for streaming/real-time media.
* **Payload Length:** 16-bit unsigned integer. The size of the payload in octets, including any extension headers.
* **Next Header:** 8-bit selector.  Identifies the type of header immediately following the IPv6 header.
* **Hop Limit:** 8-bit unsigned integer. This is same as TTL in IPv4.

####  IPv6 Extension Headers (EH)
* IPv4 Options perform a very important role in the IP protocol operation therefore the capability had to be preserved in IPv6.
The functionality of options is removed from the main header and implemented through a set of additional headers called extension headers.
* The main header remains fixed in size (40 bytes) while customized EHs are added as needed.