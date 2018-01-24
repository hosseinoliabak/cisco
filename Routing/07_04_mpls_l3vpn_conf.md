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
</pre>