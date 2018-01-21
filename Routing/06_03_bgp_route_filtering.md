# BGP route filtering
* BGP allows the filtering of BGP Update messages on any BGP router. (In OSPF we could only have filtering on ABRs or ASBRs)
* The router can filter updates *per neighbor* (not per interface) for both inbound and outbound Update on any BGP router.
* BGP matches the NLRI but can also match the large set of BGP Path Attributes (PA).
* The filters mut apply to specific neighbors with BGP. (BGP configuration does not allow filtering of all inbound or outbound updates.)

Why BGP filtering?
* As a service provider perspective we don't need filtering.
* As a customer perspective we need because if we don't filter, our AS becomes a transit AS.
  * In this case, we permit out public IP address to pass but we block any other addresses.

Scenario:

As we can see in previous example, although in R4 the route is selected from ISP2 (which we expect),
But let's remove R2 from the topology.
<pre>
R4#<b>show ip bgp</b>
BGP table version is 23, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  12.12.12.0/24    2.2.2.2                  0             0 64500 i
 * i                  5.5.5.5                  0    100      0 64500 i
 *>  13.13.13.0/24    2.2.2.2                                0 64500 i
 * i                  5.5.5.5                  0    100      0 64500 i
 * i 45.45.45.0/24    5.5.5.5                  0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 <s>*   160.160.160.0/24 2.2.2.2                                0 64500 64520 ?</s>
 *>i                  5.5.5.5                  0    100      0 64520 ?
</pre>
We only permit public IP address block from R2 and R3 to R4, R5, and R6.

For R2, I show how to filter using prefix-list. in R3 I show how to filter using a route-map
<pre>
R2(config)#<b>ip prefix-list public_only seq 10 permit 12.12.12.0/24</b>
R2(config)#<b>ip prefix-list public_only seq 20 permit 13.13.13.0/24</b>
R2(config)#<b>router bgp 64500</b>
R2(config-router)#<b>neighbor 4.4.4.4 prefix-list public_only out</b>
R2(config-router)#<b>neighbor 5.5.5.5 prefix-list public_only out</b>
</pre> 
As we can see below, in R4 we no longer have path from `2.2.2.2` to reach R6's LAN
<pre>
R4#<b>show ip bgp</b>
BGP table version is 31, local router ID is 4.4.4.4
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  12.12.12.0/24    2.2.2.2                  0             0 64500 i
 * i                  5.5.5.5                  0    100      0 64500 i
 *>  13.13.13.0/24    2.2.2.2                                0 64500 i
 * i                  5.5.5.5                  0    100      0 64500 i
 * i 45.45.45.0/24    5.5.5.5                  0    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>i 160.160.160.0/24 5.5.5.5                  0    100      0 64520 ?
</pre>

Here is another type of verification:

Before route filtering:
<pre>
R2(config-router)#<b>do show ip bgp neighbor 4.4.4.4 advertised-routes</b>
BGP table version is 5, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  12.12.12.0/24    0.0.0.0                  0         32768 i
 r>i 13.13.13.0/24    1.1.1.1                  0    100      0 i
 *>i 160.160.160.0/24 3.3.3.3                  0    100      0 64520 ?

Total number of prefixes 3 
</pre>
After route filtering:
<pre>
R2(config-router)#<b>do show ip bgp neighbor 4.4.4.4 advertised-routes</b>
BGP table version is 5, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  12.12.12.0/24    0.0.0.0                  0         32768 i
 r>i 13.13.13.0/24    1.1.1.1                  0    100      0 i

Total number of prefixes 2 
</pre>

Let's go configure R3 using a route-map.
Before we continue, let's see what prefixes R3 advertises to R6
<pre>
R3(config-router)#<b>do sho ip bgp nei 6.6.6.6 adv</b>        
BGP table version is 15, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 12.12.12.0/24    1.1.1.1                  0    100      0 i
 *>  13.13.13.0/24    0.0.0.0                  0         32768 i
 <s>*>i 45.45.45.0/24    2.2.2.2                  0    100      0 64510 i</s>
</pre>
Let's allow R3 only advertises `12.12.12.0/24` and `13.13.13.0/24` not `45.45.45.0/24`
<pre>
R3(config)#<b>ip prefix-list public_only seq 10 permit 13.13.13.0/24</b>
R3(config)#<b>ip prefix-list public_only seq 20 permit 12.12.12.0/24</b>
R3(config)#<b>route-map bgp_public_only permit</b> 
R3(config-route-map)#<b>match ip address prefix-list public_only</b>
R3(config-router)#<b>neighbor 6.6.6.6 route-map bgp_public_only out</b>
</pre>
Verification:
<pre>
R3#<b>show ip bgp neighbors 6.6.6.6 advertised-routes</b> 
BGP table version is 15, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 12.12.12.0/24    1.1.1.1                  0    100      0 i
 *>  13.13.13.0/24    0.0.0.0                  0         32768 i

Total number of prefixes 2 
</pre>
