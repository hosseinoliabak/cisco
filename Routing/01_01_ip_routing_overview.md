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

### Address Types:
Like IPv4, IPv6 has unicast and multicast addressing support. In this course for now,
I am concerning to unicast address types.

#### Unicast Address Types (RFC 4291)
* Global: AKA Aggregatable Global Unicast Addresses
  * Routable on the public Internet
  * Begin with the 2000::/3 prefix
* Link-local
  * Unique to a link but NOT necessarily unique to a router
  * Not routable
  * Begin with fe80::/64 prefix (in practice)
* Unspecified Address ::/128
  * Never assigned to an Interface
  * Used as a source address in the absence of an IPv6 address

### How are the IPv6 unicast addresses assigned?
#### 1. Static
<pre>
Router(config)#<b>interface gigabitEthernet 0/0</b>
Router(config-if)#<b>ipv6 address 2001:db9:45:c0de::1/64</b>
Router(config-if)#<b>ipv6 address fe80::14:1 link-local</b></pre>
<pre>
Router#<b>show ipv6 interface gigabitEthernet 0/0</b>
GigabitEthernet0/0 is up, line protocol is up
  <b>IPv6 is enabled, link-local address is FE80::14:1</b>
  No Virtual link-local address(es):
  Global unicast address(es):
    <b>2001:DB9:45:C0DE::1, subnet is 2001:DB9:45:C0DE::/64</b>
  Joined group address(es):
    FF02::1
    FF02::1:FF00:1
    FF02::1:FF14:1
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds (using 30000)
  ND NS retransmit interval is 1000 milliseconds
</pre>
#### 2. Modified EUI-64
* Can be used to generate *global unicast* and the *link-local* addresses
* A /64 prefix is manually specified
* The remaining 64 bits of the address are generated automatically
#### 3. DHCPv6
* Client/Server architecture
* Uses UDP multicasts instead of broadcasts
* Controls address assignments (leases)
* Gives DNS settings and other options
* Uses UDP multicasts instead of broadcasts
* DHCP servers receive messages using link-scope multicast
* DHCP relay for when server is not on the same link

##### 3.1. Stateless DHCPv6
* DHCPv6 server doesn't assign IP addresses to the clients 
* Clients will have IPs through autoconfiguration process 
* DHCPv6 assigns other information such ad DNS, DGW to the clients 

<pre>
R1(config)#<b>ipv6 dhcp pool MYPOOL</b> #Create an IPv6 pool, even though we don't be assigning IPv6 addresses to hosts 
R1(config-dhcpv6)#<b>dns-server 2001:DB8:21:5555:5</b> #Specify the information for a DNS server that the auto configured host should use. 
R1(config)#<b>int fa 0/1</b> #On the interface connecting to the client we specify which DHCP pool to reference (we may have several). The IPv6 address is already assigned. 
R1(config-if)#<b>ipv6 dhcp server MYPOOL</b> #We can append rapid-commit to this command for rapid mode 
R1(config-if)#<b>ipv6 nd other-config-flag</b> #In the router advertisement, set the "O" flag (means other options are available via DHCPv6 Lite server) to on. This will let the auto configured clients know there is other or optional DHCP information they can ask for. 
</pre>

##### 3.2. Stateful DHCPv6
* Assigning IPv6 addresses to clients 
* DHCPv6 knows which IP assigned to what client 
* DHCPv6 does not allow you to exclude addresses as you can with DHCPv4 
* DHCPv6 does not allow you to configure manual address bindings 
<pre>
R1(config)#<b>ipv6 dhcp pool STATEFUL-POOL</b> #Creates an IPv6 pool 
R1(config)#<b>address prefix 2001:DB8:21::/64</b> #similar to command network in DHCPv4 
R1(config-dhcpv6)#<b>dns-server 2001:DB8:21:5555:5</b> #Specify the information for a DNS server 
R1(config)#<b>int fa 0/1</b> #On the interface connecting to the client we specify which DHCP pool to reference (we may have several) 
R1(config-if)#<b>ipv6 dhcp server STATEFUL-POOL</b> #We can append rapid-commit to this command for rapid mode 
R1(config-if)#<b>ipv6 nd managed-config-flag</b> #indicates that hosts must use R1 as the DHCP server 
R1#<b>show ipv6 dhcp pool</b> 
R1#<b>show ipv6 dhcp binding</b> #display current DHCPv6 bindings 
R1#<b>debug ipv6 dhcp</b> 
R1#<b>clear ipv6 dhcp binding</b> #Manually clear an address binding 
</pre>

#### Router as an stateful DHCPv6 client 
<pre>
R1(config)#<b>interface FastEthernet 0/1</b>
R1(config-if)#<b>ipv6 enable</b> #to enable IPv6  on the interface 
R1(config-if)#<b>ipv6 address dhcp</b> tells the router to get IP from DHCP server. We can append rapid-commit to this command for rapid mode 
</pre>

#### Configuring a DHCPv6 Relay Agent
<pre>
R1(config-if)#<b>ipv6 dhcp relay destination dhcp_server_ipv6</b> #like ip helper-address dhcp_server_ipv4 
</pre>

#### 4. Stateless address auto-configuration (SLAAC)
* ARP in IPv4 is replaced by ICMPv6 ND (Neighbor Discovery) with 4 different types of communication in 2 major categories, Solicitations (S): asking for something; Advertisements (A): advertising periodically or responding to something (triggered immediately)
* Neighbor Diecovery Protocol (NDP) - RFC 4861
  * Similar to IPv4 processes such as ARP and ICMP redirect
  * SLAAC uses it to determine network prefix and default gateway information
  * Detecting duplicate IPv6 addresses
  * L3 to L3 address resolution (similar to ARP in IPv6)
  * ICMPv6 message types used by NDP
    * Router Solicitation (RS)
      * Sent by a network device when it needs network prefix, prefix length, or default GW
      * Router Solicitation, Who is the router on the LAN?
      * Sent to the link-local all-routers multicast ff02::2
    * Router Advertisement (RA)
      * every 200 seconds by default or by replying router solicitation
      * the neighbor is saying I am the router on the LAN. 
      * if there is an IPv6 router on the link, then it will send to the link-local
      * all-nodes multicast address ff02::1 with prefix information
    * Neighbor Solicitation (NS)
      * like ARP request, asking information about neighbor; what is your MAC address?
      * Address resolution
      * Duplicate Address Detection (DAD): DAD is also done even if we use DHCPv4 and EUI-64
    * Neighbor Advertisement (NA)
      * is the reply, the neighbor is saying this is my MAC address 
      
* Can uses EUI-64 to generate the interface ID portion of the address
* Prefix is automatically determined from router advertisements

#### Configuration
* Example 1
<pre>
R1#<b>debug ipv6 nd</b>
R1(config)#<b>interface gigabitEthernet 0/1</b>
R1(config-if)#<b>ipv6 enable</b>
*Dec  7 20:26:36.752: ICMPv6-ND: (GigabitEthernet0/1) L2 came up
*Dec  7 20:26:36.753: IPv6-Addrmgr-ND: DAD request for FE80::227:1DFF:FE03:CC01 on GigabitEthernet0/1
*Dec  7 20:26:36.755: ICMPv6-ND: (GigabitEthernet0/1,FE80::227:1DFF:FE03:CC01) Sending DAD NS [Nonce: a135.3950.428c]
*Dec  7 20:26:36.757: ICMPv6-ND: ND output feature SEND executed on 3 - rc=0

*Dec  7 20:26:37.756: IPv6-Addrmgr-ND: DAD: FE80::227:1DFF:FE03:CC01 is unique.
*Dec  7 20:26:37.759: ICMPv6-ND: (GigabitEthernet0/1,FE80::227:1DFF:FE03:CC01) Sending NA to FF02::1
*Dec  7 20:26:37.760: ICMPv6-ND: (GigabitEthernet0/1) L3 came up
*Dec  7 20:26:37.763: ICMPv6-ND: (GigabitEthernet0/1,FE80::227:1DFF:FE03:CC01) Linklocal Up
*Dec  7 20:26:37.764: ICMPv6-ND: ND output feature SEND executed on 3 - rc=0

R1#<b>show interfaces gigabitEthernet 0/1 | in bia</b>
  Hardware is iGbE, address is <b>0027.1d03.cc01</b> (bia 0027.1d03.cc01)
R1#<b>show ipv6 interface gigabitEthernet 0/1</b>
GigabitEthernet0/1 is up, line protocol is up
  <b>IPv6 is enabled, link-local address is FE80::227:1DFF:FE03:CC01</b>
  No Virtual link-local address(es):
  No global unicast address is configured
  Joined group address(es):
    FF02::1
    FF02::1:FF03:CC01
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds (using 30000)
  ND NS retransmit interval is 1000 milliseconds
</pre>
* Example 2 requirements:
  * Assign the unicast address 2001:db8:14::1/64 to R1's gig0/1 interface
  * Configure R2 to use SLAAC to receive an IPv6 address from R1

<pre>
R1(config)#<b>ipv6 unicast-routing</b>
R1(config)#<b>interface gigabitEthernet 0/1</b>
R1(config-if)#<b>ipv6 address 2001:db8:14::1/64</b>
R1(config-if)#<b>no shutdown</b></pre>

<pre>
R2(config)#<b>interface gigabitEthernet 0/1</b>
R2(config-if)#<b>ipv6 address autoconfig</b>
R2(config-if)#<b>no shutdown</b></pre>
* Verification:
<pre>
R1#<b>show ipv6 interface brief</b>
GigabitEthernet0/0     [administratively down/down]
    unassigned
GigabitEthernet0/1     [up/up]
    FE80::227:1DFF:FE03:CC01
    <b>2001:DB8:14::1</b>
</pre>
<pre>
R2#<b>show ipv6 interface brief</b>
GigabitEthernet0/0     [administratively down/down]
    unassigned
GigabitEthernet0/1     [up/up]
    FE80::227:1DFF:FE9F:4D01
    <b>2001:DB8:14:0:227:1DFF:FE9F:4D01</b>
</pre>

### IPv6 Network Prefix Translation (NPTv6)
* Unique Local Addresses (RFC 4193)
  * Use the **fd00::/8** prefix with the rest of the address randomly generated
  * Not routable in public Internet
  * But routable internally
  * Why not use global unicast on all devices?
    * Global unicast addresses may be allocated to an Internet service provider
    * Switching providers readdressing network devices
  * NPTv6 creates statless, 1:1 mappings between global unicast and unique local addresses
  * The global unicast and unique local prefixes differ, but the rest of the address stays the same

## IPv4 and IPv6 Interoperability
* IPv6 and IPv4 can coexists
  * **Dual Stack:** The host runs IPv4 and IPv6 simultaneously.
    * The entire network between them has to support IPv6
  * **Tunneling IPv6 over IPv4:** Allows *some* devices on the network to run IPv6. IPv6 packets will be encapsulated by IPv4 header
    * Tunneling does not provide communication between an IPv4-only host and an IPv6-only host
  * **Stateful NAT-64:** Allows IPv4-only host to talk to IPv6-only host
    * e.g. IPv4 Address 10.1.1.2 <-> IPv6 address 2001::2
    * Stateful: One-to-One translation
  * **Stateless NAT-64:**
    * Uses embedded IPv4-in-IPv6 address instead of static mappings
    * Example: **192.1.1.1** <-> 2001:DB9:0:1::**C001:101**
      * C0010101 is the hexadecimal format of 192.1.1.1
