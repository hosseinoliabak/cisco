# Open Shortest Path First (OSPF)
* OSPF's complexity is due to its scalability
* OSPF routers sends LSU to each others. LSU contains different types of LSAs
* Each OSPF router creates a Router LSA (Type-1 LSA) containing the information of:
  * Operational interface
  * Associated cost
  * Connected IP network
  * `show ip ospf database router`
  * In type-1 LSA: **LSID = RID**
  * Type-1 LSA link types:
    * Type 1: point-to-point: no information about routing but only used for SPF graph
    * Type 2: Transit network
    * Type 3: Stub network
    * Type 4: Virtual link: similar to point-to-poit
* Each OSPF router *floods* its LSA to connected routers
* All OSPF routers end up with the link-state database (LSDB)
  * LSDB contains different types of LSAs
  * `show ip ospf database`
* With this map of the network, OSPF routers use Dijkstra shortest path first (SPF) algorithm
to arrive at the same routing decisions
  * `show ip route ospf`
  * `show ip ospf`: shows howmany time SPF algorithm is run for each area
* OSPFv2 has 7 different LSA types with varying amounts of information for serving in different purposes:
  * This is for scalability to minimize the size of LSDB
  * *OSPF areas* limit the propagation of LSAs
* OSPF routers ensure they have the latest information in their LSDB by the sequence number field
inside the LSAs. When a router receives an LSA, first it checks its LSDB to see if it has already information
about that LSA. If it doesn't have, it simply updates its LSDB; but if it has, here is where sequence
number comes to play.
  * If sequence numbers are equal -> new LSA is ignored
  * If the new LSA's sequence number is less than the sequence number of LSA in LSDB -> New LSA is ignored
    * Then the router sends an LSU containing its LSA in the LSDB to the sender saying that "you have old information, here is newer LSA"
  * If the new LSA's sequence number is greater than what it has in its LSDB, the routers updates its LSDB then sends an LSACK to the sender,
  then it updates its routing table and floods the new LSA
  * After that there is no LSA flooding in the area unless in the event of topology change
    * But keep in mind OSPF routers send LSA summary in every 30 minutes.
* OSPF areas are implemented in a 2-level hierarchy
  * The area 0 backbone connects to all other areas
  * All other areas whch are logically connected to area 0
  * Reduces the number of redundant paths
  * Some LSAs stay within an area while others are flooded to all routers
* To see the neighbors: `show ip ospf neighbors`
* To see interfaces participating in OSPF: `show ip ospf interface`

### DR, BDR, DROTHER in multi access networks
* For fast convergence
* When a router senses a change in topology, it sends the change (Type-1 LSA) to 224.0.0.6 (DR and BDR listening to)
* Then DR sends the LSU (containing Type-2 LSA) to all OSPF routers (224.0.0.5)
  * `show ip ospf database network` to see Type-2 LSA
  * In type-2 LSA: **LSID = IP<sub>DR</sub>**
* The routers which are not DR nor BDR are DROTHERS
* DR and BDR is chosen by:
  1. bigger OSPF interface priority is DR: `(config-if)#ip ospf priority [0-255]` `#show ip ospf neighbor`
  2. Router ID (RID): bigger is DR `router-id`
  3. Biggest loopback interface IP (weather participating in OSPF or not)
  4. Biggest physical up/up interface IP address no matter if it participates in OSPF or not
* If any of these changes and we want to apply the change, we have to use `#clear ip ospf process`
* Type-1 and Type2- LSAs stay within the area

## Virtual Link
* If areas are not connected physically to each other like below example:
* `(Rx)----Area0----(R1)----Area1----(R2)----Area2----(R3)----Area3----(R4)`
* We have to connect them together logically
* Transit area shouldn't be stub
<pre>
<b>R1</b>(config-router)# <b>area 1 virtual-link 2.2.2.2</b>
<b>R2</b>(config-router)# <b>area 1 virtual-link 1.1.1.1</b>
<b>R2</b>(config-router)# <b>area 2 virtual-link 3.3.3.3</b>
<b>R3</b>(config-router)# <b>area 2 virtual-link 2.2.2.2</b></pre>
* `show ip ospf database`: Do not age bit = 0, to prevent LSA reflooding every 30 minutes
* How to change reference bandwidth:
  * `(config-router)#auto-cost reference-bandwidth <1-4294967>` Mib/sec
* We can change the cost at the interface level
  * `(config-if)#ip ospf cost <1-65535>`
  * `show ip ospf interface gig0/0`
## OSPF Metric
* metric = BW<sub>refrence</sub>/BW<sub>interface</sub>

## OSPF Area Types
* Normal area: Contain LSA types 1, 2, 3, 4, and 5
* Stub area
* Totally stubby area
* Not-so-stubby area (NSSA)
* Totally not-so-stubby area

### Type-3 LSA
* Is made by ABR
* Carries subnets listed by Type-1 and Type-2 LSA from one area to another along with their costs
  * Does NOT carry topology information
* Is not flooded to Totally stubby and totally NSSA as these areas are Cisco proprietary
* `show ip ospf database summary`
* Cost = Cost<sub>LSA-1 or LSA-2</sub> + Cost<sub>LSA-3</sub>

### Type-4 LSA
