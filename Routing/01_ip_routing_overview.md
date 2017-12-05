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
* **IP Header Length:**  tells us the number of 32 -bit words forming the header,
the minimum length of an IP header is 20 bytes so with 32-bit increments, we would see value of 5 in this field.
