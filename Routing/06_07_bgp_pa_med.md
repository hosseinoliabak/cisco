# BGP Path Attributes - Multi-Exit Discriminator (MED)
* An enterprise has much less control over inbound routes (routes for packets coming back toward the Enterprise)
because these inbound routes exist on routers that the enterprise does not own.
* MED is a tool that allow some control over the last ASN hop between an ISP and their
enterprise customer.
* MED is originally worked for a dual-homed design.
  * We have 1 ISP but multiple link to the ISP.
  * We want to tell the ISP which link we prefer than the others.
  * Remember we don't own ISP routers. We want to advertise MED to tell the ISP which path we prefer.
* The lower MED is the better
* Default is 0
* Configuration is done by route-map

**Our scenario**
Currently R4 and R5 both choose R2 to reach network 12.12.12.0/24.
<pre>
R5#<b>show ip bgp 12.12.12.0/24 longer-prefixes</b>
BGP table version is 26, local router ID is 5.5.5.5

     Network          Next Hop            Metric LocPrf Weight Path
 * i 12.12.12.0/24    4.4.4.4                  0    100      0 64500 i
 *<b>>                   2.2.2.2</b>                  0             0 64500 i
</pre>

Now we want ISP1 send us traffic over the upper link, meaning that R5 also has to send the
traffic to R4. But we don't manage ISP routers. We want to configure R2 which we have access to.

![image](https://user-images.githubusercontent.com/31813625/35203045-2223faec-fef4-11e7-953e-ce43fcd12f60.png)
 
<pre>
R2(config)#<b>ip prefix-list PrefixMED seq 10 permit 12.12.12.0/24</b>
R2(config)#<b>ip prefix-list NET13 permit 13.13.13.0/24</b>
R2(config)#<b>route-map RmMED10 permit 10</b>  
R2(config-route-map)#<b>match ip address prefix-list PrefixMED</b>
R2(config-route-map)#<b>set metric 10</b>
R2(config-route-map)#<b>exit</b>
R2(config)#<b>route-map RmMED20 permit 10</b>                         
R2(config-route-map)#<b>match ip address prefix-list PrefixMED</b>
R2(config-route-map)#<b>set metric 20</b>
R2(config-route-map)#<b>exit</b>
R2(config)#<b>route-map RmMED20 permit 20</b>       
R2(config-route-map)#<b>match ip address prefix-list NET13</b> 
R2(config-route-map)#<b>exit</b> 
R2(config)#<b>ip prefix-list NET13 permit 13.13.13.0/24</b>
R2(config)#<b>route-map RmMED10 permit 20</b>           
R2(config-route-map)#<b>match ip address prefix-list NET13</b>
R2(config)#<b>router bgp 64500</b>
R2(config-router)#<b>neighbor 4.4.4.4 route-map RmMED10 out</b>
R2(config-router)#<b>neighbor 5.5.5.5 route-map RmMED20 out</b>
R2(config-router)#<b>end</b>
R2#<b>clear ip bgp 4.4.4.4 soft</b>
R2#<b>clear ip bgp 5.5.5.5 soft</b></pre>
<pre>
R5#<b>show ip bgp 12.12.12.0/24 longer-prefixes</b>
BGP table version is 31, local router ID is 5.5.5.5

     Network          Next Hop            Metric LocPrf Weight Path
 *<b>>i 12.12.12.0/24    4.4.4.4                 10</b>    100      0 64500 i
 *                    2.2.2.2                 20             0 64500 i
</pre>