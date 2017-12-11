# IP Prefix List
* Like an ACL for network prefixes
* Processes sequentially
* For example like:<br />
  `ip prefix-list PL1 seq 5 permit 10.1.1.0/24`<br />
  `ip prefix-list PL1 seq 10 deny 0.0.0.0/0 le 32`
* `ge` means minimum prefix length
* `le` means maximum prefix length
* IP prefix list can be used to filter LSAs. We will see this later
* IP prefix list can be applied using **route maps**

# Route Map
* A set of sequentially processed rules that can `match` certain criteria and `set` attributes.
* Example:
  <pre>
  R1(config)#<b>route-map RM1 permit 10</b>
  R1(config-route-map)#<b>match ip address prefix-list PL1</b>
  R1(config-route-map)#<b>set metric-type type-1</b></pre>

### Configuration Example.
See the example on 04_03_redistribution.md.
<pre>
R1#<b>show ip route ospf</b>
Gateway of last resort is not set

O <b>E2</b>  1.0.0.0/8 [110/20] via 10.10.1.2, 00:01:51, GigabitEthernet0/0
</pre>
We are asked to advertise the network 1.1.0.0 which have subnet mask minimum /24 and maximum /30 as E1 as opposed to E2 using route-map
<pre>
ASBR(config)#<b>ip prefix-list PL1 seq 10 permit 1.0.0.0/8 ge 24 le 30</b>
ASBR(config)#<b>route-map CONN->OSPF permit</b>
ASBR(config-route-map)#<b>match ip address prefix-list PL1</b>
ASBR(config-route-map)#<b>set metric-type 2</b>
ASBR(config-route-map)#<b>exit</b>
ASBR(config)#<b>route-map CONN->OSPF permit 20</b>
ASBR(config-route-map)#<b>exit</b>
ASBR(config)#<b>router ospf 1</b>
ASBR(config-router)#<b>redistribute connected subnets route-map CONN->OSPF</b>
ASBR(config-router)#<b>no summary-address 1.0.0.0 255.0.0.0</b></pre>
I issued `no summary-address 1.0.0.0 255.0.0.0` to override the changes of the last example
<pre>
R1#<b>show ip route ospf</b>
Gateway of last resort is not set

      1.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
O <b>E1</b>     1.1.1.0/30 [110/21] via 10.10.1.2, 00:00:09, GigabitEthernet0/0
O <b>E1</b>     1.1.2.0/24 [110/21] via 10.10.1.2, 00:00:09, GigabitEthernet0/0
O E2     1.3.0.0/16 [110/20] via 10.10.1.2, 00:00:09, GigabitEthernet0/0
</pre>