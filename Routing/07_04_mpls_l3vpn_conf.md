# Configuring MPLS L3VPN

## Configuring VRFs
<pre>
PE2(config)#ip vrf CUSTOMER-A
PE2(config-vrf)#rd 2345:1     
PE2(config-vrf)#route-target import 2345:1
PE2(config-vrf)#route-target export 2345:1
PE2(config-vrf)#ip vrf CUSTOMER-B
PE2(config-vrf)#rd 2345:2                 
PE2(config-vrf)#route-target import 2345:2
PE2(config-vrf)#route-target export 2345:2
PE2(config-vrf)#do show ip vrf
  Name                             Default RD            Interfaces
  CUSTOMER-A                       2345:1                
  CUSTOMER-B                       2345:2
PE2(config)#int gigabitEthernet 0/1.12
PE2(config-subif)#ip vrf forwarding CUSTOMER-A
PE2(config-subif)#encapsulation dot1Q 12
PE2(config-subif)#ip address 192.168.12.2 255.255.255.0
PE2(config-subif)#int gigabitEthernet 0/0.22
PE2(config-subif)#ip vrf forwarding CUSTOMER-B
PE2(config-subif)#encapsulation dot1Q 22               
PE2(config-subif)#ip address 192.168.22.2 255.255.255.0
PE2(config-subif)#do sho ip vrf
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
PE1(config)#ip bgp new-format 
PE1(config)#router bgp 2345
PE1(config-router)#no bgp default ipv4-unicast 
PE1(config-router)#neighbor 5.5.5.5 remote-as 2345
PE1(config-router)#neighbor 5.5.5.5 update-source loopback 0
PE1(config-router)#address-family vpnv4
PE1(config-router-af)#neighbor 5.5.5.5 activate 
PE1(config-router-af)#neighbor 5.5.5.5 send-community both
</pre>
<pre>
PE2(config)#ip bgp new-format 
PE2(config)#router bgp 2345
PE2(config-router)#neighbor 2.2.2.2 remote-as 2345
PE2(config-router)#neighbor 2.2.2.2 update-source loopback 0
PE2(config-router-af)#neighbor 2.2.2.2 act
PE2(config-router-af)#neighbor 2.2.2.2 send-community both
</pre>