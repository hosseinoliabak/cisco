# Zone-Based Policy Firewall (ZBPFW)
* The earlier Cisco IOS firewall feature was called Context-Based Access Control (CBAC)
which applied policies through inspect statements and configured access control
lists (ACL) between interfaces. The security domains were router interfaces
* The Zone-Based Policy Firewall (ZBPFW) is the newer Cisco implementation
of a router-based firewall that runs in Cisco IOS Software. The security domains are called <i>security zones</i>
  * A given interface belongs to one zone and one zone only
* **Cisco Common Classification Policy Language (C3PL)** consists of:
  * Class maps: Identify traffic at OSI Layer 3 through 7
  * Policy maps: Define policy for traffic at Layer 3 through 7
  * Service policies: Activate policy maps for the flows on specific direction of zone pairs

### Class map:
* To identify the traffic
* By using a match criteria. The match could be based on:
  * ACL
  * Specific protocol
  * Even we can match on another class map
* Each class map can have more than one match statement
* Logical AND/OR statements are accomplished using the match type (match-all/match-any)
* Classes within a policy are evaluated in order of configuration
* We can have class maps for inspect and QoS. Here we use class maps to "inspect"
* Syntax:
    <pre>
    R1(config)#<b>class-map type inspect match-all MyMAP</b>
    R1(config-cmap)#<b>match access-group 101</b>
    R1(config-cmap)#<b>match protocol http</b>
    R1(config-cmap)#<b>exit</b></pre>

### Policy map:
* The place that class maps are applied
* Possible actions a policy map can take:
  * Inspect (to create a stateful enty for the traffic)
  * Permit: If we permit, we are not creaing a stateful entry
  * Drop
  * Log
* Unclassified traffic matches the default class, referred to as class-default
* The default action is to drop the traffic
* Default action can be customized per policy map to implement appropriate default actions
* Syntax:
    <pre>
    R1(config)#<b>policy-map type inspect MyPOLICY</b>
    R1(config-pmap)#<b>class type inspect MyMAP</b>
    R1(config-pmap-c)#<b>inspect</b>
    R1(config-pmap-c)#<b>exit</b>
    R1(config-pmap)#<b>exit</b></pre>

### Zones

* **Zone Pairs** are grouped source/destination pairs to facilitate policy assignment
  * Separate resources into zones based on resource sensitivity and trustworthiness
  * Interfaces in the same zone can communicate without restriction
  * All traffic is denied between zones
  * A zone pair allows you to specify a unidirectional firewall policy between two security zones
* The self-zone is system-defined and applies to traffic to or from the router itself
  * The self zone controls traffic sent to the router itself or originated by the router
  * Unless you specify a zone-pair combining self zone with another zone, all traffic
  from that zone sent to the router itself is allowed (The router is not protected)
  * To control traffic that the router can send into a zone use a zone-pair from self to another zone.
  Use inspect in the service-policy to allow the return traffic
  * To filter the traffic that the router can accept, use a zone-pair from
  another zone to self. Only the packets accepted by this zone-pair's
  service-policy will be accepted by the router
* Syntax:
<pre>
R1(config)#<b>zone security INSIDE</b>
R1(config-sec-zone)#<b>zone security OUTSIDE</b>
R1(config)#<b>interface gigabitEthernet 0/0</b>
R1(config-if)#<b>zone-member security OUTSIDE</b>
R1(config-)#<b>interface gigabitEthernet 0/1</b>
R1(config-if)#<b>zone-member security INSIDE</b>
R1(config)#<b>zone-pair security IN>OUT source INSIDE destination OUTSIDE</b>
R1(config-sec-zone-pair)#<b>service-policy type inspect MyPOLICY</b>
R1(config-sec-zone-pair)#<b>exit</b></pre>