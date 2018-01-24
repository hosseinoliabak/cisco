# Configuring MPLS L3VPN

## Configuring VRFs
<pre>
PE2(config)#<b>ip vrf CUSTOMER-A</b>
PE2(config-vrf)#<b>rd 2345:1</b>     
PE2(config-vrf)#<b>route-target import 2345:1</b>
PE2(config-vrf)#<b>route-target export 2345:1</b>
PE2(config-vrf)#<b>ip vrf CUSTOMER-B</b>
PE2(config-vrf)#<b>rd 2345:2</b>                 
PE2(config-vrf)#<b>route-target import 2345:2</b>
PE2(config-vrf)#<b>route-target export 2345:2</b>
PE2(config-vrf)#<b>do show ip vrf</b>
  Name                             Default RD            Interfaces
  CUSTOMER-A                       2345:1                
  CUSTOMER-B                       2345:2
PE2(config)#<b>int gigabitEthernet 0/1.12</b>
PE2(config-subif)#<b>ip vrf forwarding CUSTOMER-A</b>
PE2(config-subif)#<b>encapsulation dot1Q 12</b>
PE2(config-subif)#<b>ip address 192.168.12.2 255.255.255.0</b>
PE2(config-subif)#<b>int gigabitEthernet 0/0.22</b>
PE2(config-subif)#<b>ip vrf forwarding CUSTOMER-B</b>
PE2(config-subif)#<b>encapsulation dot1Q 22</b>               
PE2(config-subif)#<b>ip address 192.168.22.2 255.255.255.0</b>
PE2(config-subif)#<b>do sho ip vrf</b>
  Name                             Default RD            Interfaces
  CUSTOMER-A                       2345:1                Gi0/1.12
  CUSTOMER-B                       2345:2                Gi0/0.22
PE1#<b>ping vrf CUSTOMER-A 192.168.12.1</b>
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/7/9 ms
PE1#<b>show ip route vrf CUSTOMER-A</b>

Routing Table: CUSTOMER-A
Gateway of last resort is not set

      192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.12.0/24 is directly connected, GigabitEthernet0/1.12
L        192.168.12.2/32 is directly connected, GigabitEthernet0/1.12
</pre>
You have to configure PE2 as well. I don't put the configuration here. But at the end, I will put
`show run` of all devices.

## Configuring MP-iBGP
<pre>
PE1(config)#<b>ip bgp new-format</b> 
PE1(config)#<b>router bgp 2345</b>
PE1(config-router)#<b>no bgp default ipv4-unicast</b> 
PE1(config-router)#<b>neighbor 5.5.5.5 remote-as 2345</b>
PE1(config-router)#<b>neighbor 5.5.5.5 update-source loopback 0</b>
PE1(config-router)#<b>address-family vpnv4</b>
PE1(config-router-af)#<b>neighbor 5.5.5.5 activate</b> 
PE1(config-router-af)#<b>neighbor 5.5.5.5 send-community both</b>
</pre>
<pre>
PE2(config)#<b>ip bgp new-format</b> 
PE2(config)#<b>router bgp 2345</b>
PE2(config-router)#<b>no bgp default ipv4-unicast</b>
PE2(config-router)#<b>neighbor 2.2.2.2 remote-as 2345</b>
PE2(config-router)#<b>neighbor 2.2.2.2 update-source loopback 0</b>
PE1(config-router)#<b>address-family vpnv4</b>
PE2(config-router-af)#<b>neighbor 2.2.2.2 act</b>
PE2(config-router-af)#<b>neighbor 2.2.2.2 send-community both</b>
</pre>

<pre>
PE1#<b>show ip bgp vpnv4 all summary</b> 
BGP router identifier 2.2.2.2, local AS number 2345
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
5.5.5.5         4         2345       4       2        1    0    0 00:00:07        0
</pre>

<pre>
PE2#<b>show ip bgp vpnv4 all summary</b>
BGP router identifier 5.5.5.5, local AS number 2345
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2.2.2.2         4         2345       2       4        1    0    0 00:00:11        0
</pre>





















