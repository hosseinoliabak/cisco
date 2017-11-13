# Protecting the Spanning Tree Protocol Topology
## STP BPDUGuard:
BPDU Guard is usually configured along with PortFast.

<img src="https://user-images.githubusercontent.com/31813625/32732992-26adafdc-c85c-11e7-9958-7079c962c715.png" width="314" height="169" />

On Fa1/0/11 we had a PC and the port was configured as a PortFast; now
we are going to unplug the PC and connect a switch, the newly added switch
will send BPDUs but we don't want to receive the BPDUs on this port
<pre>
SW1(config-if)#<b>spanning-tree bpduguard enable</b>
*Mar  1 02:38:14.528: %SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port Fa1/0/11 with BPDU Guard enabled. Disabling port.
*Mar  1 02:38:14.528: %PM-4-ERR_DISABLE: bpduguard error detected on Fa1/0/11, <b>putting Fa1/0/11 in err-disable state</b>
</pre>
<pre>
SW1#<b>show interfaces status err-disabled</b>

Port      Name               Status            Reason               Err-disabled Vlans
Fa1/0/11                     <b>err-disabled  bpduguard</b>
</pre>
## STP Root Guard:
RootGuard will make sure you don’t accept a certain switch as a root
bridge. BPDUs are sent and processed normally but if a switch suddenly
sends a BPDU with a superior bridge ID you won’t accept it as the root
bridge.

The rootguard feature is implemented on a per interface basis with the
`spanning-tree guard root` command, and the feature essentially prevents
the interface from ever becoming a root port by placing the interface
into root-inconsistent state upon receiving a superior BPDU.  When the
superior BPDU goes away, the port automatically transitions back to
forwarding.
### Configuration and verification example:
On the STP root bridge [designated] interfaces:
<pre>
Distribution_1(config)#<b>interface range fastEthernet 1/0/23 - 24</b>
*Mar  1 01:27:59.479: %SPANTREE-2-ROOTGUARD_CONFIG_CHANGE: Root guard enabled on port FastEthernet1/0/23.
</pre>
<pre>
Distribution_1#show spanning-tree interface fastEthernet 1/0/23 detail
 Port 25 (FastEthernet1/0/23) of VLAN0001 is broken  (Root Inconsistent)
   Port path cost 19, Port priority 128, Port Identifier 128.25.
   Designated root has priority 32769, address 001d.707b.4200
   Designated bridge has priority 32769, address 001d.707b.4200
   Designated port id is 128.25, designated path cost 0
   Timers: message age 2, forward delay 0, hold 0
   Number of transitions to forwarding state: 1
   Link type is point-to-point by default
   <b>Root guard is enabled on the port</b>
   BPDU: sent 2663, received 48
</pre>
Now I made Distribution_2 as the STP root bridge, then interfaces Fa1/0/23:
anf Fa1/0/24 go to ROOT_Inc state.
<pre>
Distribution_1#<b>show spanning-tree interface fastEthernet 1/0/23</b>

Vlan                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
VLAN0001            Desg BKN*19        128.25   P2p <b>*ROOT_Inc</b>
</pre>
<pre>
Distribution_1#<b>show spanning-tree interface fastEthernet 1/0/24</b>

Vlan                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
VLAN0001            Desg BKN*19        128.26   P2p <b>*ROOT_Inc</b>
</pre>
<pre>
Distribution_1#<b>show spanning-tree inconsistentports</b>

Name                 Interface                Inconsistency
-------------------- ------------------------ ------------------
VLAN0001             FastEthernet1/0/<b>23</b>       <b>Root Inconsistent</b>
VLAN0001             FastEthernet1/0/<b>24</b>       <b>Root Inconsistent</b>
</pre>
Now I am going to suppress the superior BPDU by giving a higher priority
to Distribution_2. Then we will issue `show spanning-tree inconsistentports`
again on Distribution_1
<pre>
Distribution_1#<b>show spanning-tree inconsistentports</b>

Name                 Interface                Inconsistency
-------------------- ------------------------ ------------------

Number of inconsistent ports (segments) in the system : 0
</pre>I