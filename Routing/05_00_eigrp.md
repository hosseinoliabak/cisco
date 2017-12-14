# Enhanced Interior Gateway Routing Protocol (EIGRP)
* EIGRP is an enhanced distance vector protocol.
* Does not keep link state information about every router.
* Instead each router shares its own routes with adjacent neighbors (directly connected).
* In RIP we only have one table which is the *routing table*.
In OSPF we have 3 tables. *routing table*, *neighbor's table*, *topology or database table*.
In EIGRP we are also having these 3 tables but different terminology and information.
* IP protocol 88
* EIGRP group address: 224.0.0.10
* EIGRP uses *Reliable Transport Protocol (RTP)* to ensure packets are sent in-order.
* EIGRP Route Types:
  * Internal: Originate from within EIGRP AS. Administrative distance 90.
  * External: Redistributed into the EIGRP AS. Administrative Distance 170.

### The diffusing Update Algorithm (DUAL)
* The goal is to find the shortest path to a destination without loop
* In OSPF, all routers know about every link state in the routing domain which easily avoids routing loops
* But in EIGRP loop is prevented by concept of
  * Feasible Distance (FD): distance from router to the destination with lowest metric
  * Reported Distance (RD): each router reports its own distance to the destination
  * Successor (S): The path which will put into the routing table
  * Feasible Successor (FS): if Feasible Condition met (see next line), then FS would be the backup path
  * Feasible Condition: RD<sub>FS</sub> < FD<sub>S</sub>
* The purpose of having a feasible successor:
  * Provides a pre-computed, loop-free path to the destination prefix if the successor route goes down
  * There can be multiple feasible successors

### EIGRP Packet Types:
* **Hello**
  * To discover EIGRP neighbors
    * Most networks: unreliably multicast every 5 seconds
    * NBMA networks: Unicast every 60 seconds
  * Every hello packet includes a hold-time which tells the **receiving neighbor** how often to expect hello messages
    * Defaults to 3x the Hello interval
    * So, hello interval is locally significant, but hold-time is significant for the neighbor
    * `passive-interface` prevents eigrp sending hello on that interface; however, EIGRP still advertises the subnets belonging to that interface.
      * Used for stub networks
* **Update**
  * Conveys routing prefix and metric information
  * Non-periodic: Not sent at defined intervals
  * Partial Update: Only changed routing information is sent
  * Bounded: Only routers that need routing updates receive them
* **Acknowledgment**
  * Really just unicast hello packets
  * Used to confirm receipt of a reliably transmitted packet
* **Query**
  * If there is no FS, the router which encountered a failed route:
    * sets the failed route to an *active* state
    * it sets the feasible distance to *infinity*
    * it sends a query (with cost of *infinity*) message over its links
    * When a query is sent to a neighbor, that neigbor has 3 minutes to reply (*Active Timer*)
 * **Reply**
  * Then is the neighbor has a route to the destination, it sends a *reply* to the router which sent a *query* message
    * *reply* contains the cost of itself to the destination which in the query was asked for
    * Then the router which sent the query insalls the router which sent the reply as its new successor
    * Finally Both routers send *update* packet to their neighbors telling about the new distance to the destination
      * And the router which lost the rout, now consider the path to the destination as a *passive* state
        * A route is in the *passive* state once the DUAL algorithm has converged on a final cost
* Stuck in Active (SIA)
  * When a route goes active, queries are sent to neighbors to find the best bath to the prefix
  * If the active timer (default: 3 minutes) expires before all replies are received, the route becomes SIA
  * Adjacencies with the neighbors that did not respond are reset
  * **EIGRP stubs** are designed to avoid routes getting stuck in active
    * Stub routers do not advertise learned routes from one neighbor to other EIGRP neighbors
    * the nonstub routers do not send Query messages to the stub routers
    * The eigrp stub command has several options. **connected** and **summary** option are the defaults
      * **connected :** Advertise connected routes but only for interfaces matched with a network command.
      * **summary :** Advertise auto-summarized or statically configured summary routes.
      * **static :** Advertise static routes, assuming that the redistribute static command is configured.
      * **leak-map** *name* **:** Advertise routes (that would otherwise be part of a summary route) specified by a leak map.
      * **redistributed:** Advertise redistributed routes, assuming that redistribution is configured.
      * **receive-only:** Does not advertise any routes. This option cannot be used with any other option.
    * Note that stub routers still form neighborships, even in receive-only mode. The stub
router simply performs less work and reduces the Query scope because neighbors will
not send these routers any Query messages.
* Default route and propagating it in EIGRP
  * in ospf we had `(config-router)#default-information originate` but in EIGRP we can do
  `(config-router)#redistribute static` then write a default static route to the destination for example
  `ip route 0.0.0.0 0.0.0.0 gig 0/0`
  * Another option is, if we issued command `network 0.0.0.0` and the default route, there would be no need
  to `redistribute static` eigrp subcommand

### EIGRP Wighted Metric Formula
* K1, K2: related to Bandwidth
* K3: related to Delay
* K4, K5: related to Reliability
* Default values K1 and K3 are set to 1, all others are set to 0
* By default the EIGRP metric would be <pre>256 (10<sup>7</sup>/Bandwidth in Kibps + Delay in microseconds/10)</pre>

### Traditional EIGRP and Named EIGRP Configurations Compared

#### Basic configuration comparison
##### Traditional Approach
<pre>
R1(config)#<b>router eigrp 1</b>
R1(config-router)#<b>network 0.0.0.0</b></pre>

##### Named Approach
<pre>
R1(config)#<b>router eigrp R1</b>
R1(config-router)#<b>address-family ipv4 autonomous-system 1</b>
R1(config-router-af)#<b>network 0.0.0.0</b></pre>

* **The Named EIGRP Hierarchical Structure**
* Traditional and Named Approaches are compatible
* 3 different modes
  * **Address-Family:** General EIGRP configuration commands are issued under this
configuration mode. For example, router ID, network, and EIGRP
stub router configurations are performed here. Multiple address
families (for example, IPv4 and IPv6) can be configured under the
same EIGRP virtual instance.
  * **Address-Family-Interface:**
  Commands entered under interface configuration mode with
a traditional EIGRP configuration are entered here for Named
EIGRP configuration. For example, timer and passive interface
configurations are performed here.
  * **Address-Family-Topology:** Commands that have a direct impact on a router’s EIGRP topology
table are given in this configuration mode. For example, variance and
redistribution are configured in this mode.

* Example: advanced EIGRP Configuration Using the Named Configuration Approach

<pre>
!
hostname R1
!
ipv6 unicast-routing

interface Loopback0
 ip address 192.168.1.1 255.255.255.0
 ipv6 address 2001::1/64
!
interface GigabitEthernet0/0
 ip address 10.10.12.1 255.255.255.252
 ipv6 enable
!
router eigrp R1
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface default
   hello-interval 2
   hold-time 10
   passive-interface
  exit-af-interface
  !
  af-interface GigabitEthernet0/0
   no passive-interface
  exit-af-interface
  !
  topology base
   variance 2
  exit-af-topology
  network 0.0.0.0
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 2
  !
  topology base
   variance 2
  exit-af-topology
 exit-address-family
</pre>