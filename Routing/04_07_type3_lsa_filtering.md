# Type 3 LSA Filtering
* We can only filter either on ABR or ASBR.
* Remember that OSPF routers don't advertise *routes*, they advertises *LSA*s.
* So we can filter LSAs
* On the other hand, in one area, all routers have to have the same LSDB otherwise they cannot run the SPF algorithm
  * So OSPF does NOT allow filtering within an area (intra-area)
  * But it allows filtering between different areas (inter area)
    * Inter-area in OSPF has distance-vector logic

<img src="https://user-images.githubusercontent.com/31813625/33811789-24ddb0a0-dde5-11e7-8bb8-eb353052826c.png" width="506" height="425" />

* We have two choices:
  * To filter subnet 3 from being advertised on ABR1 and filter subnets 2 and 3 from being advertised
  using `filter-list`
    * `area 0 filter-list prefix ACL out`
      * Because are filtering from area 0 to 1 (`out`), we see the impact on all routers in area 1
      * We could also use `area 1` and `in`
  * To filter OSPF Routes Added to the Routing Table using `distribute-list`


## Using a filter-list

OSPF basic configuration of the routers

![image](https://user-images.githubusercontent.com/31813625/33861276-02b0fae8-deab-11e7-927b-14a142b63e01.png)


**B1**
<pre>
router ospf 1
 auto-cost reference-bandwidth 1000
 redistribute connected subnets
 network 172.16.1.0 0.0.0.255 area 1
</pre>
**WAN1**
<pre>
router ospf 1
 auto-cost reference-bandwidth 1000
 network 10.10.0.0 0.0.255.255 area 0
 network 172.16.1.0 0.0.0.255 area 1
</pre>
**WAN2**
<pre>
router ospf 1
 auto-cost reference-bandwidth 1000
 network 10.10.0.0 0.0.255.255 area 0
 network 172.16.1.0 0.0.0.255 area 1
</pre>
Let's see the routing table of B1 before filtering
<pre>
B1#<b>show ip route ospf</b>
Gateway of last resort is 172.16.1.3 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 172.16.1.3, 00:02:25, GigabitEthernet0/0
                [110/1] via 172.16.1.2, 00:02:15, GigabitEthernet0/0
      10.0.0.0/30 is subnetted, 8 subnets
O IA     10.10.10.0 [110/3] via 172.16.1.3, 00:01:11, GigabitEthernet0/0
                    [110/3] via 172.16.1.2, 00:00:54, GigabitEthernet0/0
O IA     10.10.11.0 [110/2] via 172.16.1.2, 00:00:54, GigabitEthernet0/0
<b>O IA     10.10.12.0 [110/2] via 172.16.1.3, 00:01:11, GigabitEthernet0/0
                    [110/2] via 172.16.1.2, 00:00:54, GigabitEthernet0/0
O IA     10.10.12.4 [110/2] via 172.16.1.2, 00:00:54, GigabitEthernet0/0
O IA     10.10.12.8 [110/2] via 172.16.1.3, 00:01:11, GigabitEthernet0/0
O IA     10.10.12.12 [110/3] via 172.16.1.3, 00:01:11, GigabitEthernet0/0
                     [110/3] via 172.16.1.2, 00:00:54, GigabitEthernet0/0</b>
O IA     10.10.20.0 [110/3] via 172.16.1.3, 00:01:11, GigabitEthernet0/0
                    [110/3] via 172.16.1.2, 00:00:54, GigabitEthernet0/0
O IA     10.10.22.0 [110/2] via 172.16.1.3, 00:01:11, GigabitEthernet0/0
</pre>
Now let's do the filtering configuration so that we remove all networks starting with 10.10.12.X
advertised by WAN2 from the routing table of B1

<pre>
WAN2(config)#<b>ip prefix-list filter-from-area-0 seq 10 deny 10.10.12.0/24 ge 30 le 30</b>
WAN2(config)#<b>ip prefix-list filter-from-area-0 seq 20 permit 0.0.0.0/0 le 32</b>
WAN2(config)#<b>router ospf 1</b>
WAN2(config-router)#<b>area 0 filter-list prefix filter-from-area-0 out</b></pre>

<pre>
B1#<b>show ip route ospf</b>
Gateway of last resort is 172.16.1.3 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 172.16.1.3, 00:12:22, GigabitEthernet0/0
                [110/1] via 172.16.1.2, 00:12:12, GigabitEthernet0/0
      10.0.0.0/30 is subnetted, 8 subnets
O IA     10.10.10.0 [110/3] via 172.16.1.3, 00:01:33, GigabitEthernet0/0
                    [110/3] via 172.16.1.2, 00:10:51, GigabitEthernet0/0
O IA     10.10.11.0 [110/2] via 172.16.1.2, 00:10:51, GigabitEthernet0/0
<b>O IA     10.10.12.0 [110/2] via 172.16.1.2, 00:10:51, GigabitEthernet0/0
O IA     10.10.12.4 [110/2] via 172.16.1.2, 00:10:51, GigabitEthernet0/0
O IA     10.10.12.8 [110/3] via 172.16.1.2, 00:01:57, GigabitEthernet0/0
O IA     10.10.12.12 [110/3] via 172.16.1.2, 00:10:51, GigabitEthernet0/0</b>
O IA     10.10.20.0 [110/3] via 172.16.1.3, 00:01:33, GigabitEthernet0/0
                    [110/3] via 172.16.1.2, 00:10:51, GigabitEthernet0/0
O IA     10.10.22.0 [110/2] via 172.16.1.3, 00:01:33, GigabitEthernet0/0
</pre>

## Filtering OSPF Routes Added to the Routing Table
* In this method LSDB still doesn't change, but routes will be removed from routing table
<pre>
B1(config)#<b>ip prefix-list filter-from-area-0 seq 10 deny 10.10.12.0/24 ge 30 le 30</b>
B1(config)#<b>ip prefix-list filter-from-area-0 seq 20 permit 0.0.0.0/0 le 32</b>
B1(config-router)#<b>distribute-list prefix filter-from-area-0 in</b></pre>

<pre>
B1#<b>show ip route ospf</b>
Gateway of last resort is 172.16.1.3 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 172.16.1.3, 00:00:28, GigabitEthernet0/0
                [110/1] via 172.16.1.2, 00:00:28, GigabitEthernet0/0
      10.0.0.0/30 is subnetted, 4 subnets
O IA     10.10.10.0 [110/3] via 172.16.1.3, 00:00:28, GigabitEthernet0/0
                    [110/3] via 172.16.1.2, 00:00:28, GigabitEthernet0/0
O IA     10.10.11.0 [110/2] via 172.16.1.2, 00:00:28, GigabitEthernet0/0
O IA     10.10.20.0 [110/3] via 172.16.1.3, 00:00:28, GigabitEthernet0/0
                    [110/3] via 172.16.1.2, 00:00:28, GigabitEthernet0/0
O IA     10.10.22.0 [110/2] via 172.16.1.3, 00:00:28, GigabitEthernet0/0
</pre>