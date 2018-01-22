# BGP Cisco-proprietary Weight Settings
* To influence an enterprise's outbound route.
* A Cisco router can use BGP Weight on that single router to influence that one router's
choice of outbound route. 
* 0-65,5353 (Defaults: 0 for learned routes, 32768 for locally injected routes)
* We cannot change the defaults
* Configuration
  * per prefix: route-map
  * per neighbor (all routes learned from this neighbor): neighbor weight

1. Per prefix
* Scenario: In our BGP configuration, R2 selected R4 to reach 45.45.45.0/24.
We want to change it using `weight` so that R2 chooses R5 to reach 45.45.45.0/24.

<pre>
R2#<b>show ip bgp 45.45.45.0/24</b>
BGP routing table entry for 45.45.45.0/24, version 4
Paths: (2 available, best #2, table default)
  Advertised to update-groups:
     11        
  Refresh Epoch 2
  64510
    5.5.5.5 from 5.5.5.5 (5.5.5.5)
      Origin IGP, metric 0, localpref 100, valid, external
      rx pathid: 0, tx pathid: 0
  Refresh Epoch 1
  64510
    4.4.4.4 from 4.4.4.4 (4.4.4.4)
      Origin IGP, metric 0, localpref 100, valid, external, <b>best</b>
      rx pathid: 0, tx pathid: 0x0
</pre>
<pre>
R2(config)#<b>ip prefix-list weight50 seq 10 permit 45.45.45.0/24</b>
R2(config)#<b>route-map RmW50 permit</b>
R2(config-route-map)#<b>match ip address prefix-list weight50</b>
R2(config-route-map)#<b>set weight 50</b>
R2(config)#<b>route-map RmW50 permit 20</b>
R2(config-route-map)#<b>exit</b>
R2(config)#<b>router bgp 64500</b>
R2(config-router)#<b>neighbor 5.5.5.5 route-map RmW50 in</b>
R2#<b>clear ip bgp 5.5.5.5 soft</b> # Because we only want to enable the weight of the incoming update from R5
R2#<b>show ip bgp 45.45.45.0/24</b>
BGP routing table entry for 45.45.45.0/24, version 6
Paths: (2 available, best #1, table default)
  Advertised to update-groups:
     11        
  Refresh Epoch 3
  64510
    5.5.5.5 from 5.5.5.5 (5.5.5.5)
      Origin IGP, metric 0, localpref 100, <b>weight 50</b>, valid, external, <b>best</b>
      rx pathid: 0, tx pathid: 0x0
  Refresh Epoch 1
  64510
    4.4.4.4 from 4.4.4.4 (4.4.4.4)
      Origin IGP, metric 0, localpref 100, valid, external
      rx pathid: 0, tx pathid: 0
</pre>
2. Per neighbor:
* Scenario:By now, the weight of path to R5 is 50 and R4 is 0. Let's change the 
weight of R4 to 60 to become the best path for R2 to reach 45.45.45.0/24
<pre>
R2(config)#<b>router bgp 64500</b> 
R2(config-router)#<b>neighbor 4.4.4.4 weight 60</b> 
R2(config-router)#<b>end</b> 
R2#<b>clear ip bgp 4.4.4.4 soft</b> 
R2#<b>show ip bgp 45.45.45.0/24 longer-prefixes</b> 
BGP table version is 8, local router ID is 2.2.2.2
     Network          Next Hop            Metric LocPrf Weight Path
 *   45.45.45.0/24    5.5.5.5                  0            <b>50</b> 64510 i
 *<b>></b>                   <b>4.4.4.4</b>                  0            <b>60</b> 64510 i
</pre>
Note that when we change the wight in this method (per neighbor), we don't have control
for what subnets we want to change the weight. Here, not only R2 reaches to 45.45.45.0/24
from R4, but R2 also reaches to 160.160.160.0/24 through R4.
<pre>
R2#<b>show ip bgp</b>
     Network          Next Hop            Metric LocPrf Weight Path
 *>  12.12.12.0/24    0.0.0.0                  0         32768 i
 * i                  1.1.1.1                  0    100      0 i
 r i 13.13.13.0/24    3.3.3.3                  0    100      0 i
 r>i                  1.1.1.1                  0    100      0 i
 *   45.45.45.0/24    5.5.5.5                  0            50 64510 i
 *>                   4.4.4.4                  0            60 64510 i
 * i <b>160.160.160.0/24</b> 3.3.3.3                  0    100      0 64520 ?
 *                    5.5.5.5                                0 64510 64520 ?
<b> *>                   4.4.4.4                               60 64510 64520 ?</b></pre>
