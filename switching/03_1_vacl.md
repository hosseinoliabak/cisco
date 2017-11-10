# VLAN ACLs (VACLs)
Unlike Cisco IOS ACLs that are applied on routed packets only, VACLs apply to all packets and can be applied to any VLAN.
VACLs are processed in the ACL TCAM hardware.

You can configure VACLs for IP and MAC-layer traffic.


## Example:
```
Switch(config)#access-list 101 permit ip host 192.168.122.73 host 192.168.122.115
Switch(config)#access-list 101 permit ip host 192.168.122.115 host 192.168.122.73

Switch(config)#vlan access-map FILTER-DESKTOP-LAPTOP 10
Switch(config-access-map)#match ip address 101 
Switch(config-access-map)#action drop
Switch(config-access-map)#exit
Switch(config)#vlan access-map FILTER-DESKTOP-LAPTOP 20
Switch(config-access-map)#action forward 
Switch(config-access-map)#exit
Switch(config)#vlan filter FILTER-DESKTOP-LAPTOP vlan-list 1
```

### To verify:
```
SW2#show vlan filter
VLAN Map FILTER-DESKTOP-LAPTOP is filtering VLANs:
  1


SW2#show vlan access-map FILTER-DESKTOP-LAPTOP
Vlan access-map "FILTER-DESKTOP-LAPTOP"  10
  Match clauses:
    ip  address: 100
  Action:
    drop
Vlan access-map "FILTER-DESKTOP-LAPTOP"  20
  Match clauses:
  Action:
    forward

```