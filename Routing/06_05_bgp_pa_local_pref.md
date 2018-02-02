# BGP Path Attributes
Unlike IGPs, BGP needs to choose one and only one route as best, in part
because BGP advertises only best routes to its neighbor.

**BGP Best Path Algorithm:**
0. Next hop: is reachable? if not, that path will be set aside.
1. Bigger Weight: It is not PA. It is a Cisco feature. Since it is not a PA so it is not advertises
in BGP updates. Hence the scope is on inbound route updates and influences only that one router's choice.
2. Bigger LOCAL_PREF
3. Locally injected routes: locally injected is better than iBGP/eBGP
4. shorter AS_PATH (default in BGP)
5. Origin: i>e>?
  * i: network command. In aggregation if as_set is not used; if as_set is used and all subnets use origin code `i`. and `neighbor default-originate` command 
  * e: egp (no longer exists)
  * ?: redistribution. In aggregation if as_set is used and at least one subnets uses origin code `?`. and `default information originate` command
6. Smaller MED: 
7. Neighbor type: eBGP > iBGP
8. Smaller IGP metric to next hop: 
9. Oldest eBGP route
10. Lowest neighbor BGP RID
11. Lowest neighbor IP address

Some of the BGP best path steps purposefully give the engineer a tool for influencing
the choice of best path.
* Popular to influence outbound routes
  * Weight
  * Local preference
  * AS_PATH
* Popular to influence inbound routes
  * MED

# Local_pref
* It is a path attribute. Hence we can advertise it (not to eBGP but to iBGP).
* gives the routers inside a single AS a value that they can set per-route and advertise to
all iBGP routers inside the AS to agree about the best exit point.
* As our AS routers decides the exit points it is in iBGP scope not in eBGP.
* Default is 100, the higher the better.
* We can change the default using `bgp default local-preference <0-4294967295>` BGP subcommand.
* Configuration: `neighbor route-map` command with `in` option for updates from an eBGP peer.

### Scenario
In our scenario, the best AS to work on is 64500 where we have R2 and R3 as exit points.
Currently R2 is selected to reach network 45.45.45.0/24 for R1 and R3.
<pre>
R1#<b>show ip bgp 45.45.45.0/24 longer-prefixes</b> 
BGP table version is 14, local router ID is 1.1.1.1
     Network          Next Hop            Metric LocPrf Weight Path
 *<b>>i 45.45.45.0/24    2.2.2.2</b>                  0    100      0 64510 i
</pre>
<pre>
R3#<b>show ip bgp 45.45.45.0/24 longer-prefixes</b>
BGP table version is 15, local router ID is 3.3.3.3
     Network          Next Hop            Metric LocPrf Weight Path
 *<b>>i 45.45.45.0/24    2.2.2.2</b>                  0    100      0 64510 i
 *                    6.6.6.6                                0 64520 64510 i
</pre>
R3 wants that R1 and R2 agree that it is better preference to reach network 45.45.45.0/24.
<pre>
R3(config)#<b>ip prefix-list Local_pref_200 seq 10 permit 45.45.45.0/24</b>
R3(config)#<b>route-map RmLP200 permit 10</b>
R3(config-route-map)#<b>match ip address prefix-list Local_pref_200</b>
R3(config-route-map)#<b>set local-preference 200</b>
R3(config-route-map)#<b>exit</b>
R3(config)#<b>route-map RmLP200 permit 20</b>                
R3(config-route-map)#<b>exit</b>
R3(config)#<b>router bgp 64500</b>
R3(config-router)#<b>neighbor 6.6.6.6 route-map RmLP200 in</b>
R3(config-router)#<b>end</b>
R3#<b>clear ip bgp 6.6.6.6 soft</b> 
</pre>
Let's verify on R1
<pre>
R1#<b>show ip bgp 45.45.45.0/24 longer-prefixes</b> 
BGP table version is 15, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 <b>*>i 45.45.45.0/24    3.3.3.3</b>                  0    <b>200</b>      0 64520 64510 i
 * i                  2.2.2.2                  0    100      0 64510 i
</pre>
Let's see the result in R2.
In R2 weight is 60 (configured in the previous 06_04 file). Local preference is 200.
Because *weight* is before *local preference* in the bestpath selection order, R2 chose
R4 to reach 45.45.45.0/24
<pre>
R2#<b>show ip bgp 45.45.45.0/24 longer-prefixes</b> 
BGP table version is 8, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 45.45.45.0/24    3.3.3.3                  0    <b>200</b>      0 64520 64510 i
 *                    5.5.5.5                  0            50 64510 i
 <b>*>                   4.4.4.4</b>                  0            <b>60</b> 64510 i
</pre>


