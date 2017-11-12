# Configuring Spanning Tree PortFast, UplinkFast, and BackboneFast
## PortFast:
PortFast causes a switch or trunk port to enter the spanning tree
forwarding state immediately, bypassing the listening and learning states.

You can use PortFast to connect a single end station or a switch port to
a switch port. If you enable PortFast on a port that is connected to
another Layer 2 device, such as a switch, you might create network loops.
### PortFast Configuration Example:
We can enable PortFast using one of these 2 methods:
<pre>
SW3(config)#<b>spanning-tree portfast default</b>
%Warning: this command enables portfast by default on all interfaces. You
 should now disable portfast explicitly on switched ports leading to hubs,
 switches and bridges as they may create temporary bridging loops.
SW3(config)#
SW3(config)#<b>interface range gigabitEthernet 1/0/1 - 2</b>
SW3(config-if-range)#<b>no spanning-tree portfast</b>
</pre>
<pre>
SW3(config)#<b>interface fastEthernet 1/0/19</b>
SW3(config-if)#<b>spanning-tree portfast</b>
%Warning: portfast should only be enabled on ports connected to a single
 host. Connecting hubs, concentrators, switches, bridges, etc... to this
 interface  when portfast is enabled, can cause temporary bridging loops.
 Use with CAUTION

%Portfast has been configured on FastEthernet1/0/19 but will only
 have effect when the interface is in a non-trunking mode.
</pre>
To verify:
<pre>
SW3#<b>show spanning-tree interface fastEthernet 1/0/19 portfast</b>
VLAN0001         <b>enabled</b>
</pre>
<pre>
SW3#<b>show spanning-tree detail</b>

 VLAN0001 is executing the ieee compatible Spanning Tree protocol
  Bridge Identifier has priority 32768, sysid 1, address 0022.be5a.0680
  Configured hello time 2, max age 20, forward delay 15
  We are the root of the spanning tree
  Topology change flag not set, detected flag not set
  Number of topology changes 0 last change occurred 00:01:01 ago
  Times:  hold 1, topology change 35, notification 2
          hello 2, max age 20, forward delay 15
  Timers: hello 0, topology change 0, notification 0, aging 300

 Port 21 <b>(FastEthernet1/0/19)</b> of VLAN0001 is forwarding
   Port path cost 19, Port priority 128, Port Identifier 128.21.
   Designated root has priority 32769, address 0022.be5a.0680
   Designated bridge has priority 32769, address 0022.be5a.0680
   Designated port id is 128.21, designated path cost 0
   Timers: message age 0, forward delay 0, hold 0
   Number of transitions to forwarding state: 1
   <b>The port is in the portfast mode</b>
   Link type is point-to-point by default
   BPDU: sent 31, received 0
</pre>
## UplinkFast
**Note:** Note UplinkFast is most useful in wiring-closet switches that
have a limited number of active VLANs. This enhancement might not be
useful for other types of applications and should not be enabled on
backbone or distribution layer switches.

**Here is an example how UplinkFast works:**
If Access to Distribution_1 link disconnected, we want to bring the
link Access to Distribution_2 link up as quick as possible
<img src="https://user-images.githubusercontent.com/31813625/32703803-2dfa5b16-c7c9-11e7-9b26-3ca502557448.png" width="298" height="271" />