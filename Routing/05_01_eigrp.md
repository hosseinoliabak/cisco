# IEGRP Named Configuration Approach Example

![image](https://user-images.githubusercontent.com/31813625/33921238-8f85d278-df90-11e7-994c-734140680c63.png)

* requirements:
  * All interfaces should participate in EIGRP for IPv4 AS 1.
  * All interfaces should participate in EIGRP for IPv6 AS 2.
  * The variance option should be set to 2 for both autonomous systems.
  * The Hello Interval should be set to 2 seconds for EIGRP for IPv4 AS1.
  * The Hold Time should be set to 10 seconds for EIGRP for IPv4 AS1.
<pre>
R1(config)#<b>router eigrp R1</b>
R1(config-router)#<b>address-family ipv4 unicast autonomous-system 1</b>
R1(config-router-af)#<b>network 0.0.0.0</b>
R1(config-router-af)#<b>af-interface default</b>
R1(config-router-af-interface)#<b>hello-interval 2</b>
R1(config-router-af-interface)#<b>hold-time 10</b>
R1(config-router-af-interface)#<b>passive-interface</b>
R1(config-router-af-interface)#<b>exit-af-interface</b>
R1(config-router-af)#<b>af-interface gigabitEthernet 0/0</b>
R1(config-router-af-interface)#<b>no passive-interface</b>
R1(config-router-af-interface)#<b>exit-af-interface</b>
R1(config-router-af)#<b>topology base</b>
R1(config-router-af-topology)#<b>variance 2</b>
R1(config-router-af-topology)#<b>exit-af-topology</b>
R1(config-router-af)#<b>exit-address-family</b>
R1(config-router)#<b>address-family ipv6 unicast autonomous-system 2</b>
R1(config-router-af)#<b>topology base</b>
R1(config-router-af-topology)#<b>variance 2</b>
R1(config-router-af-topology)#<b>exit-af-topology</b>
R1(config-router-af)#<b>exit-address-family</b>
</pre>

I am going to configure R2 in traditional aproach
<pre>
R2(config)#<b>router eigrp 2</b>
R2(config-router)#<b>variance 2</b>
R2(config-router)#<b>network 0.0.0.0</b>
R2(config-router)#<b>passive-interface default</b>
R2(config-router)#<b>no passive-interface gigabitEthernet 0/0</b>
R2(config)#<b>ipv6 router eigrp 2</b>
R2(config-rtr)#<b>variance 2</b>
R2(config)#<b>interface loopback 0</b>
R2(config-if)#<b>ipv6 eigrp 2</b>
R2(config)#<b>interface gigabitEthernet 0/0</b>
R2(config-if)#<b>ip hello-interval eigrp 1 2</b>
R2(config-if)#<b>ip hold-time eigrp 1 10</b>
R2(config-if)#<b>ipv6 eigrp 2</b>
</pre>

### Verification on R1
<pre>
R1#<b>show ip route eigrp</b>
Gateway of last resort is not set

D     192.168.2.0/24 [90/2570240] via 10.10.12.2, 00:00:22, GigabitEthernet0/0
</pre>

<pre>
R1#<b>show ipv6 route eigrp</b>
IPv6 Routing Table - default - 4 entries
D   2002::/64 [90/2570240]
     via FE80::26F:C8FF:FE76:5600, GigabitEthernet0/0
</pre>

<pre>
R1#<b>show ip eigrp neighbors</b>
EIGRP-IPv4 VR(R1) Address-Family Neighbors for AS(1)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   10.10.12.2              Gi0/0                     8 00:01:29    9   100  0  5
</pre>
* **Smooth round-trip time (SRTT):** The number of milliseconds it takes for an EIGRP packet to be sent to this neighbor and for the local router to receive an acknowledgment of that packet.
* **Retransmission timeout (RTO), in milliseconds:** The amount of time that the software waits before retransmitting a packet from the retransmission queue to a neighbor
* **Q Count:** The number of EIGRP packets (Update, Query, and Reply) that the software is waiting to send.
<pre>
R1#<b>show ipv6 eigrp neighbors</b>
EIGRP-IPv6 VR(R1) Address-Family Neighbors for AS(2)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   Link-local address:     Gi0/0                    13 00:07:01    6   100  0  3
    FE80::26F:C8FF:FE76:5600
</pre>

<pre>
R1#<b>show ip eigrp topology</b>
EIGRP-IPv4 VR(R1) Topology Table for AS(1)/ID(192.168.1.1)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status

P 192.168.2.0/24, 1 successors, FD is 328990720
        via 10.10.12.2 (328990720/327761920), GigabitEthernet0/0
P 10.10.12.0/30, 1 successors, FD is 1310720
        via Connected, GigabitEthernet0/0
P 192.168.1.0/24, 1 successors, FD is 163840
        via Connected, Loopback0
</pre>

<pre>
R1#<b>show ipv6 eigrp topology</b>
EIGRP-IPv6 VR(R1) Topology Table for AS(2)/ID(192.168.1.1)
Codes: P - Passive, A - Active, U - Update, Q - Query, R - Reply,
       r - reply Status, s - sia Status

P 2001::/64, 1 successors, FD is 163840
        via Connected, Loopback0
P 2002::/64, 1 successors, FD is 328990720
        via FE80::26F:C8FF:FE76:5600 (328990720/327761920), GigabitEthernet0/0
</pre>

<pre>
R1#<b>show ipv6 interface loopback 0</b>
EIGRP-IPv6 VR(R1) Address-Family Interfaces for AS(2)
                              Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
Interface              Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Gi0/0                    1        0/0       0/0           6       0/0           50           0
Lo0                      0        0/0       0/0           0       0/0            0           0
</pre>

<pre>
R1#<b>show ipv6 interface loopback 0</b>
Loopback0 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::26F:C8FF:FEF9:700
  No Virtual link-local address(es):
  Global unicast address(es):
    2001::1, subnet is 2001::/64
  Joined group address(es):
    FF02::1
    FF02::2
    FF02::A
    FF02::1:FF00:1
    FF02::1:FFF9:700
  MTU is 1514 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  ND DAD is not supported
  ND reachable time is 30000 milliseconds (using 30000)
  ND advertised reachable time is 0 (unspecified)
  ND advertised retransmit interval is 0 (unspecified)
  ND router advertisements live for 1800 seconds
  ND advertised default router preference is Medium
  ND RAs are suppressed (periodic)
  Hosts use stateless autoconfig for addresses.
</pre>

## EIGRP Message Authentication

R Configuration (Named Approach)
<pre>
R1(config)#<b>router eigrp R1</b>
R1(config-router)#<b>address-family ipv4 unicast autonomous-system 1</b>
R1(config-router-af)#<b>af-interface gigabitEthernet 0/0</b>
R1(config-router-af-interface)#<b>authentication key-chain KC-EIGRP</b>
R1(config-router-af-interface)#<b>authentication mode ?</b>
  <b>hmac-sha-256</b>  HMAC-SHA-256 Authentication
  <b>md5</b>           Keyed message digest
R1(config-router-af-interface)#<b>authentication mode md5</b>
*Dec 14 00:17:43.124: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.10.12.2 (GigabitEthernet0/0) is down: authentication mode changed
R1(config-router-af-interface)#<b>exit-af-interface</b>
R1(config-router-af)#<b>exit-address-family</b>
R1(config-router)#<b>exit</b>
R1(config)#<b>key chain KC-EIGRP</b>
R1(config-keychain)#<b>key 1</b>
R1(config-keychain-key)#<b>key-string VALUE</b>
R1(config-keychain-key)#<b>cryptographic-algorithm md5</b>
R1(config-keychain-key)#<b>exit</b>
R1(config-keychain)#<b>exit</b>
</pre>

R2 Configuration (Traditional Approah)
<pre>
R2(config)#<b>key chain KC-EIGRP</b>
R2(config-keychain)#<b>key 1</b>
R2(config-keychain-key)#<b>key-string VALUE</b>
R2(config-keychain-key)#<b>cryptographic-algorithm md5</b>
R2(config-keychain-key)#<b>exit</b>
R2(config-keychain)#<b>exit</b>
R2(config)#<b>interface gigabitEthernet 0/0</b>
R2(config-if)#<b>ip authentication key-chain eigrp 1 KC-EIGRP</b>
R2(config-if)#<b>ip authentication mode eigrp 1 ?</b>
  <b>md5</b>  Keyed message digest
R2(config-if)#<b>ip authentication mode eigrp 1 md5</b>
*Dec 14 00:41:17.148: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 10.10.12.1 (GigabitEthernet0/0) is up: new adjacency
</pre>

**Verification**
<pre>
R1#<b>show ip eigrp 1 interfaces detail gigabitEthernet 0/0</b>
EIGRP-IPv4 VR(R1) Address-Family Interfaces for AS(1)
                              Xmit Queue   PeerQ        Mean   Pacing Time   Multicast    Pending
Interface              Peers  Un/Reliable  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Gi0/0                    1        0/0       0/0        1604       0/0         8016           0
  Hello-interval is 2, Hold-time is 10
  Split-horizon is enabled
  Next xmit serial <none>
  Packetized sent/expedited: 5/1
  Hello's sent/expedited: 3046/4
  Un/reliable mcasts: 0/6  Un/reliable ucasts: 6/6
  Mcast exceptions: 0  CR packets: 0  ACKs suppressed: 0
  Retransmissions sent: 3  Out-of-sequence rcvd: 1
  Topology-ids on interface - 0
  <b>Authentication mode is md5,  key-chain is "KC-EIGRP"</b>
  Topologies advertised on this interface:  base
  Topologies not advertised on this interface:
</pre>