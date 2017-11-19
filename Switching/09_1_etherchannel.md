# Aggregating switch links with Etherchannel
Make sure all ports participating in Etherchannel have the same configuration
* Duplex
* Speed
* Native VLAN
* Allowed VLANs
* Switchport mode (Access/Trunk)

## PAgP and LACP Modes
* PAgP (Cisco proprietary)
* LACP (A subcomponent of IEEE 802.3ad)
### PAgP
The interface can configured as:
* <b>On:</b> interface becomes member of the etherchannel but does not negotiate
* <b>Desirable:</b> interface will actively ask the other side to become an etherchannel
* <b>Auto:</b> interface will wait passively for the other side to ask to become an etherchannel
* <b>Off:</b> no etherchannel configured on the interface

| | On | Desirable | Auto | Off |
| --- | --- | --- | --- | --- |
| __On__ | Yes | No | No | No |
| __Desirable__ | No | Yes | Yes | No |
| __Auto__ | No | Yes | No | No
| __Off__ | No | No | No | No |

<pre>
SW1#<b>debug etherchannel</b> 
PAgP/LACP Shim/FEC debugging is on
SW1(config)#<b>interface range gigabitEthernet 0/1 - 2</b>
SW1(config-if-range)#<b>channel-group 1 mode desirable</b>
Creating a port-channel interface Port-channel 1
SW1(config-if-range)#<b>channel-group 1 mode desirable</b> 
Creating a port-channel interface Port-channel 1
*Nov 19 16:44:52.994: FEC: new chnl group 1 created 
*Nov 19 16:44:52.994: FEC: add port Gi0/1 to group 1 
*Nov 19 16:44:53.019: FEC: pagp_switch_port_statechange: port_insert Po1 
*Nov 19 16:44:53.025: FEC: pagp_switch_allocate_agport: allocate agport Po1 to group 1 
*Nov 19 16:44:53.030: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/1 Po1 flag = 1 1 
*Nov 19 16:44:53.030: FEC: pagp_switch_port_attrib_diff: Gi0/1 Po1 same
*Nov 19 16:44:53.031: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:44:53.041: FEC: add port Gi0/2 to group 1 
*Nov 19 16:44:53.043: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/2 Po1 flag = 1 1 
*Nov 19 16:44:53.044: FEC: pagp_switch_port_attrib_diff: Gi0/2 Po1 same
*Nov 19 16:44:53.044: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:44:53.045: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/2 Gi0/1 flag = 1 1 
*Nov 19 16:44:53.045: FEC: pagp_switch_port_attrib_diff: compare PAgP modes for Gi0/2
*Nov 19 16:44:53.046: FEC: pagp_switch_port_attrib_diff: Gi0/2 Gi0/1 same
*Nov 19 16:44:53.046: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:44:53.047: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/2 Gi0/1 flag = 1 1 
*Nov 19 16:44:53.047: FEC: pagp_switch_port_attrib_diff: compare PAgP modes for Gi0/2
*Nov 19 16:44:53.048: FEC: pagp_switch_port_attrib_diff: Gi0/2 Gi0/1 same
*Nov 19 16:44:53.048: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:45:02.554: FEC: fec_dontbundle: Gi0/2
*Nov 19 16:45:02.798: FEC: fec_dontbundle: Gi0/1
</pre>
<pre>
SW2(config)#<b>interface range gigabitEthernet 0/1 - 2</b>
SW2(config-if-range)#<b>channel-group 1 mode auto</b> 
*Nov 19 16:47:19.354: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
</pre>
<pre>
SW1#
*Nov 19 16:47:15.567: FEC: add port (Gi0/2) to agport (Po1) 
*Nov 19 16:47:15.568: FEC: pagp_switch_add_port_to_agport_internal: msg to PM to bundle port Gi0/2 with Po1 
*Nov 19 16:47:15.569: FEC: pagp_switch_want_to_bundle: Bndl msg to PM for port Gi0/2 to Agport Po1 
*Nov 19 16:47:18.570: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
*Nov 19 16:47:19.326: FEC: add port (Gi0/1) to agport (Po1) 
*Nov 19 16:47:19.327: FEC: pagp_switch_add_port_to_agport_internal: msg to PM to bundle port Gi0/1 with Po1 
*Nov 19 16:47:19.327: FEC: pagp_switch_want_to_bundle: Bndl msg to PM for port Gi0/1 to Agport Po1 
</pre>
<pre>
SW1#<b>show etherchannel port-channel</b> 
		Channel-group listing: 
		----------------------

Group: 1 
----------
		Port-channels in the group: 
		---------------------------

Port-channel: Po1
------------

Age of the Port-channel   = 0d:00h:04m:53s
Logical slot/port   = 16/0          Number of ports = 2
GC                  = 0x00010001      HotStandBy port = null
Port state          = Port-channel Ag-Inuse 
Protocol            =   PAgP
Port security       = Disabled

Ports in the Port-channel: 

Index   Load   Port     EC state        No of bits
------+------+------+------------------+-----------
  0     00     Gi0/1    Desirable-Sl       0
  0     00     Gi0/2    Desirable-Sl       0

Time since last port bundled:    0d:00h:02m:26s    Gi0/1
</pre>
<pre>
SW1(config)#<b>interface port-channel 1</b>
*Nov 19 16:50:45.183: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/1 Gi0/2 flag = 1 1 
*Nov 19 16:50:45.184: FEC: pagp_switch_port_attrib_diff: compare PAgP modes for Gi0/1
*Nov 19 16:50:45.185: FEC: pagp_switch_port_attrib_diff: Gi0/1 Gi0/2 same
*Nov 19 16:50:45.185: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:50:45.186: FEC: pagp_switch_react_port_attrib_change: nothing changed in agport configuration
SW1(config-if)#<b>switchport mode trunk</b>
*Nov 19 16:51:24.737: FEC: pagp_switch_delete_port_from_agport_internal: delete_port_from_agport: for port Gi0/1 
*Nov 19 16:51:24.737: FEC: delete port (Gi0/1) from agport (Po1) 
*Nov 19 16:51:24.738: FEC: Un-Bndl msg NOT send to PM for port Gi0/1 from Po1 
*Nov 19 16:51:24.744: FEC: pagp_switch_delete_port_from_agport_internal: delete_port_from_agport: for port Gi0/2 
*Nov 19 16:51:24.744: FEC: delete port (Gi0/2) from agport (Po1) 
*Nov 19 16:51:24.745: FEC: Un-Bndl msg NOT send to PM for port Gi0/2 from Po1 
*Nov 19 16:51:26.748: %LINK-3-UPDOWN: Interface Port-channel1, changed state to down
*Nov 19 16:51:27.740: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/1 Po1 flag = 1 1 
*Nov 19 16:51:27.741: FEC: pagp_switch_port_attrib_diff: Gi0/1 Po1 same
*Nov 19 16:51:27.741: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:51:27.747: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to down
*Nov 19 16:51:27.751: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/2 Po1 flag = 1 1 
*Nov 19 16:51:27.752: FEC: pagp_switch_port_attrib_diff: Gi0/2 Po1 same
*Nov 19 16:51:27.753: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:51:27.753: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/2 Gi0/1 flag = 1 1 
*Nov 19 16:51:27.753: FEC: pagp_switch_port_attrib_diff: compare PAgP modes for Gi0/2
*Nov 19 16:51:27.754: FEC: pagp_switch_port_attrib_diff: Gi0/2 Gi0/1 same
*Nov 19 16:51:27.754: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:51:27.755: FEC: pagp_switch_agc_compatable: comparing GC values of Gi0/2 Gi0/1 flag = 1 1 
*Nov 19 16:51:27.755: FEC: pagp_switch_port_attrib_diff: compare PAgP modes for Gi0/2
*Nov 19 16:51:27.756: FEC: pagp_switch_port_attrib_diff: Gi0/2 Gi0/1 same
*Nov 19 16:51:27.756: FEC: pagp_switch_agc_compatable: GC values are compatable
*Nov 19 16:51:29.623: FEC: add port (Gi0/1) to agport (Po1) 
*Nov 19 16:51:29.626: FEC: pagp_switch_add_port_to_agport_internal: msg to PM to bundle port Gi0/1 with Po1 
*Nov 19 16:51:29.626: FEC: pagp_switch_want_to_bundle: Bndl msg to PM for port Gi0/1 to Agport Po1 
*Nov 19 16:51:30.516: FEC: add port (Gi0/2) to agport (Po1) 
*Nov 19 16:51:30.518: FEC: pagp_switch_add_port_to_agport_internal: msg to PM to bundle port Gi0/2 with Po1 
*Nov 19 16:51:30.518: FEC: pagp_switch_want_to_bundle: Bndl msg to PM for port Gi0/2 to Agport Po1 
*Nov 19 16:51:31.733: %LINK-3-UPDOWN: Interface Port-channel1, changed state to up
*Nov 19 16:51:32.733: %LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
</pre>
<pre>
SW1#<b>show etherchannel summary</b> 
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Gi0/1(P)    Gi0/2(P) 
</pre>
<pre>
SW1#<b>show interfaces gigabitEthernet 0/1 etherchannel</b> 
Port state    = Up Mstr In-Bndl 
Channel group = 1           Mode = Desirable-Sl    Gcchange = 0
Port-channel  = Po1         GC   = 0x00010001      Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   PAgP

Flags:  S - Device is sending Slow hello.  C - Device is in Consistent state.
        A - Device is in Auto mode.        P - Device learns on physical port.
        d - PAgP is down.
Timers: H - Hello timer is running.        Q - Quit timer is running.
        S - Switching timer is running.    I - Interface timer is running.

Local information:
                                Hello    Partner  PAgP     Learning  Group
Port      Flags State   Timers  Interval Count   Priority   Method  Ifindex
Gi0/1     SC	U6/S7   H	30s	 1        128        Any      18

Partner's information:

          Partner              Partner          Partner         Partner Group
Port      Name                 Device ID        Port       Age  Flags   Cap.
Gi0/1     SW2                  0097.8116.2900	Gi0/1       12s SAC	10001 

Age of the port in the current state: 0d:00h:02m:29s
</pre>


### LACP
The interface can configured as:
* **On:** interface becomes member of the etherchannel but does not negotiate
* **Active:** interface will actively ask the other side to become an etherchannel
* **Passive:** interface will wait passively for the other side to ask to become an etherchannel
* **Off:** no etherchannel configured on the interface

| | On | Active | Passive | Off |
| --- | --- | --- | --- | --- |
| __On__ | Yes | No | No | No |
| __Active__ | No | Yes | Yes | No |
| __Passive__ | No | Yes | No | No
| __Off__ | No | No | No | No |

The same concept as PAgP, but you are dealing with Active and Passive mode as opposed to Desirable and Auto.

In PAgP we can have up to 8 interfaces, but in LACP we can have 16 interfaces from which 8 are standby. How to determine which interfaces be Active and which be standby?
##### A: Switch priority
1. Switch priority: The switch with the lower LACP priority makes the decision. Default priority is 32768
2. If priorities are equal then switch with the lower MAC address makes the decision.
<pre>
SW1(config)#<b>lacp system-priority ?</b>
  <0-65535>  Priority value
</pre>

##### B: Port priority
We can determine the priority per ports. If the port priorities are equal, the lower 8 port numbers become active.
<pre>
SW1(config-if)#<b>lacp port-priority ?</b>
  <0-65535>  Priority value
</pre>
Default priority is 32768

### Load-balancing
To see what the default configuration is
<pre>
SW1#<b>show etherchannel load-balance</b> 
EtherChannel Load-Balancing Configuration:
        <b>src-dst-ip</b>

EtherChannel Load-Balancing Addresses Used Per-Protocol:
Non-IP: Source XOR Destination MAC address
  IPv4: Source XOR Destination IP address
  IPv6: Source XOR Destination IP address
</pre>
You can use the global `port-channel load-balance` command to change this behavior.
<pre>
SW1(config)#<b>port-channel load-balance ?</b>
  dst-ip       Dst IP Addr
  dst-mac      Dst Mac Addr
  src-dst-ip   Src XOR Dst IP Addr
  src-dst-mac  Src XOR Dst Mac Addr
  src-ip       Src IP Addr
  src-mac      Src Mac Addr
</pre>