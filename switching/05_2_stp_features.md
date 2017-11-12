# Configuring Spanning Tree PortFast, UplinkFast, and BackboneFast
## PortFast:
PortFast causes a switch or trunk port to enter the spanning tree
forwarding state immediately, bypassing the listening and learning states.

You can use PortFast to connect a single end station or a switch port to
a switch port. If you enable PortFast on a port that is connected to
another Layer 2 device, such as a switch, you might create network loops.
### PortFast Configuration Example:
We can enable PortFast using one of these 2 methods:
```
SW3(config)#spanning-tree portfast default
%Warning: this command enables portfast by default on all interfaces. You
 should now disable portfast explicitly on switched ports leading to hubs,
 switches and bridges as they may create temporary bridging loops.
SW3(config)#
SW3(config)#interface range gigabitEthernet 1/0/1 - 2
SW3(config-if-range)#no spanning-tree portfast
```
```
SW3(config)#interface fastEthernet 1/0/19
SW3(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
 host. Connecting hubs, concentrators, switches, bridges, etc... to this
 interface  when portfast is enabled, can cause temporary bridging loops.
 Use with CAUTION

%Portfast has been configured on FastEthernet1/0/19 but will only
 have effect when the interface is in a non-trunking mode.
```
To verify:
```
SW3#show spanning-tree interface fastEthernet 1/0/19 portfast
VLAN0001         enabled
```