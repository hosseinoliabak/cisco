# BGP Path Attributes - AS Path
In our scenario, R6 reaches to 12.12.12.0/24 through R3. Let's change it to R5 playing
with AS path.

I put the snapshot here again:

![bgp2](https://user-images.githubusercontent.com/31813625/35195595-d9ef54ca-fe93-11e7-9485-0eee5bb630b7.png)


<pre>
R6#<b>show ip bgp 12.12.12.0/24 longer-prefixes</b> 
BGP table version is 7, local router ID is 6.6.6.6

     Network          Next Hop            Metric LocPrf Weight Path
 *   12.12.12.0/24    5.5.5.5                                <b>0 64510 64500 i</b>
 *<b>>                   3.3.3.3                                0 64500 i</b></pre>

<pre>
R6(config)#<b>ip prefix-list ASPATH seq 10 permit 12.12.12.0/24</b>
R6(config)#<b>route-map RmASPATH permit 10</b>
R6(config-route-map)#<b>match ip address prefix-list ASPATH</b>
R6(config-route-map)#<b>set as-path prepend 64500 64500 64500</b>
R6(config-route-map)#<b>exit</b>
R6(config)#<b>route-map RmASPATH permit 20</b>               
R6(config-route-map)#<b>exit</b>
R6(config)#<b>router bgp 64520</b>
R6(config-router)#<b>neighbor 3.3.3.3 route-map RmASPATH in</b>
R6(config-router)#<b>end</b>
R6#<b>clear ip bgp 3.3.3.3 soft</b>
</pre> 
To verify:
<pre>
R6#<b>show ip bgp 12.12.12.0/24 longer-prefixes</b> 
BGP table version is 8, local router ID is 6.6.6.6

     Network          Next Hop            Metric LocPrf Weight Path
 *<b>>  12.12.12.0/24    5.5.5.5                                0 64510 64500 i</b>
 *                    3.3.3.3                                0 <b>64500 64500 64500 64500 i</b>
</pre>