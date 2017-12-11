# OSPF Route Summarization

* Only on ABR and ASBR
* in one area, all routers have to have the same LSDB otherwise they cannot run the SPF algorithm
  * So OSPF does NOT allow summarization within an area (intra-area)
  * But it allows summarization between different areas (inter area)

## Configuration

We want to advertise 10.10.1.0/25 and 10.10.1.128/25 as a 10.10.1.0/24 to R2:

![image](https://user-images.githubusercontent.com/31813625/33813525-57121496-ddf2-11e7-84aa-22e7ea668b53.png)

Let's see the routing table of R2 before summarization:
<pre>
R2#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     <b>10.10.1.1/32 [110/3] via 172.16.1.4</b>, 00:35:20, GigabitEthernet0/0
                      <b>[110/3] via 172.16.1.3</b>, 00:16:04, GigabitEthernet0/0
O IA     <b>10.10.1.129/32 [110/3] via 172.16.1.4</b>, 00:35:20, GigabitEthernet0/0
                        <b>[110/3] via 172.16.1.3</b>, 00:16:04, GigabitEthernet0/0
O IA     10.10.2.1/32 [110/3] via 172.16.1.4, 00:35:20, GigabitEthernet0/0
                      [110/3] via 172.16.1.3, 00:16:04, GigabitEthernet0/0
O IA     10.10.3.1/32 [110/3] via 172.16.1.4, 00:35:20, GigabitEthernet0/0
                      [110/3] via 172.16.1.3, 00:16:04, GigabitEthernet0/0
O IA     10.10.35.0/29 [110/2] via 172.16.1.3, 00:16:04, GigabitEthernet0/0
O IA     10.10.45.0/29 [110/2] via 172.16.1.4, 00:35:20, GigabitEthernet0/0
</pre>

**R3**
<pre>
R3(config)#<b>router ospf 1</b>
R3(config-router)#<b>area 0 range 10.10.1.0 255.255.255.0</b></pre>

Let's see the routing table of R2 after summarization on R3
<pre>
R2#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 7 subnets, 3 masks
O IA     <b>10.10.1.0/24 [110/3] via 172.16.1.3</b>, 00:00:05, GigabitEthernet0/0
O IA     <b>10.10.1.1/32 [110/3] via 172.16.1.4</b>, 00:36:11, GigabitEthernet0/0
O IA     <b>10.10.1.129/32 [110/3] via 172.16.1.4</b>, 00:36:11, GigabitEthernet0/0
O IA     10.10.2.1/32 [110/3] via 172.16.1.4, 00:36:11, GigabitEthernet0/0
                      [110/3] via 172.16.1.3, 00:16:55, GigabitEthernet0/0
O IA     10.10.3.1/32 [110/3] via 172.16.1.4, 00:36:11, GigabitEthernet0/0
                      [110/3] via 172.16.1.3, 00:16:55, GigabitEthernet0/0
O IA     10.10.35.0/29 [110/2] via 172.16.1.3, 00:16:55, GigabitEthernet0/0
O IA     10.10.45.0/29 [110/2] via 172.16.1.4, 00:36:11, GigabitEthernet0/0
</pre>
Now let's do the summarization on R4 as well.

**R4**
<pre>
R4(config)#<b>router ospf 1</b>
R4(config-router)#<b>area 0 range 10.10.1.0 255.255.255.0</b></pre>

Let's see the routing table of R2 after summarization on both R3 and R4:
<pre>
R2#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
O IA     <b>10.10.1.0/24 [110/3] via 172.16.1.4</b>, 00:10:28, GigabitEthernet0/0
                      <b>[110/3] via 172.16.1.3</b>, 00:11:50, GigabitEthernet0/0
O IA     10.10.2.1/32 [110/3] via 172.16.1.4, 00:47:56, GigabitEthernet0/0
                      [110/3] via 172.16.1.3, 00:28:40, GigabitEthernet0/0
O IA     10.10.3.1/32 [110/3] via 172.16.1.4, 00:47:56, GigabitEthernet0/0
                      [110/3] via 172.16.1.3, 00:28:40, GigabitEthernet0/0
O IA     10.10.35.0/29 [110/2] via 172.16.1.3, 00:28:40, GigabitEthernet0/0
O IA     10.10.45.0/29 [110/2] via 172.16.1.4, 00:47:56, GigabitEthernet0/0
</pre>

* Note:
  * On **ABR** We used `area X range IP MASK` under the router configuration
  * On **ASBR** we use `summary-address IP MASK` under the router configuration

One more thing:

Let's see the routing table of R3 which was one of the ABRs we did summarization on:
<pre>
R3#show ip route ospf
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 8 subnets, 3 masks
<b>O        10.10.1.0/24 is a summary, 00:00:25, Null0</b>
O        10.10.1.1/32 [110/2] via 10.10.35.5, 00:00:25, GigabitEthernet0/5
O        10.10.1.129/32 [110/2] via 10.10.35.5, 00:00:25, GigabitEthernet0/5
O        10.10.2.1/32 [110/2] via 10.10.35.5, 00:00:25, GigabitEthernet0/5
O        10.10.3.1/32 [110/2] via 10.10.35.5, 00:00:25, GigabitEthernet0/5
O        10.10.45.0/29 [110/2] via 10.10.35.5, 00:00:25, GigabitEthernet0/5
</pre>
ABR that creates the summary route will create a null0 interface to prevent loops
(If loop happened, it will continue until the IP TTL expires.).
We know that we have more specific route in the routing tables for a couple of 10.10.x.x networks.
However, when a packet received destined to 10.10.1.0 but there is no destination, the packet is sent to
blackhole null0 interface