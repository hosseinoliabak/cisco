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

## STP BPDUFilter:
The spanning-tree BPDUfilter works similar to BPDUGuard as it allows you
to block malicious BPDUs. The difference is that BPDUguard will put the
interface that it receives the BPDU on in err-disable mode while
BPDUfilter just "filters" it. In this lesson we'll take a good look at
how BPDUfilter works.

BPDUfilter can be configured globally or on the interface level and
there’s a difference:

* Global: if you enable BPDUfilter globally then any interface with
portfast enabled will not send or receive any BPDUs. When you receive a
BPDU on a portfast enabled interface then it will lose its portfast
status, disables BPDU filtering and acts as a normal interface.
* Interface: if you enable BPDUfilter on the interface it will ignore
incoming BPDUs and it will not send any BPDUs. This is the equivalent
of disabling spanning-tree.

You have to be careful when you enable BPDUfilter on interfaces.
You can use it on interfaces in access mode that connect to computers
but make sure you never configure it on interfaces connected to other
switches; if you do you might end up with a loop.

```
Switch(config)#interface fa1/0/19
Switch(config-if)#spanning-tree portfast trunk
Switch(config-if)#spanning-tree bpdufilter enable
```
Or globally:
```
Switch(config)#spanning-tree portfast bpdufilter default
```
Personally I wouldn't use this and use BPDUguard instead. If you don't
expect BPDUs on an interface then it's better to get a notification
(through err-disable) then not seeing what is going on…

## STP Root Guard:
RootGuard will make sure you don't accept a certain switch as a root
bridge. BPDUs are sent and processed normally but if a switch suddenly
sends a BPDU with a superior bridge ID you won't accept it as the root
bridge.

The rootguard feature is implemented on a per interface basis with the
`spanning-tree guard root` command, and the feature essentially prevents
the interface from ever becoming a root port by placing the interface
into root-inconsistent state upon receiving a superior BPDU.  When the
superior BPDU goes away, the port automatically transitions back to
forwarding.
### Configuration and verification example:
<img src="https://user-images.githubusercontent.com/31813625/32703803-2dfa5b16-c7c9-11e7-9b26-3ca502557448.png" width="298" height="271" />

Distribution_1 is designated as the root bridge and this is what we want.
We want to prevent the Access bridge becoming the root bridge when
something like `spanning-tree vlan 1 root primary` is issued on the
Access bridge.
We enable `spanning-tree guard root` on the interfaces facing to the Access
bridge on both Distributio_1 and Distributio_2:
<pre>
Distribution_1(config)#<b>interface fastEthernet 1/0/23</b>
Distribution_1(config-if)#<b>spanning-tree guard root</b>
*Mar  1 00:07:05.168: %SPANTREE-2-ROOTGUARD_CONFIG_CHANGE: Root guard enabled on port FastEthernet1/0/23.
</pre>
<pre>
Distribution_2(config)#<b>interface fastEthernet 1/0/21</b>
Distribution_2(config-if)#<b>spanning-tree guard root</b>
00:08:09: %SPANTREE-2-ROOTGUARD_CONFIG_CHANGE: Root guard enabled on port FastEthernet1/0/21.
</pre>
<pre>
Distribution_1#<b>show spanning-tree interface fastEthernet 1/0/23 detail</b>
 Port 25 (FastEthernet1/0/23) of VLAN0001 is broken  (Root Inconsistent)
   Port path cost 19, Port priority 128, Port Identifier 128.25.
   Designated root has priority 32769, address 001d.707b.4200
   Designated bridge has priority 32769, address 001d.707b.4200
   Designated port id is 128.25, designated path cost 0
   Timers: message age 2, forward delay 0, hold 0
   Number of transitions to forwarding state: 1
   Link type is point-to-point by default
   <span style="background-color: #FFFF00">Root guard is enabled on the port</span>
   BPDU: sent 2663, received 48
</pre>
Before issuing `spanning-tree vlan 1 root primary` command on the Access
switch, let's see the output of the `show spanning-tree` command
<pre>
Access#<b>show spanning-tree</b>

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    <span style="font-weight: bold;background-color: #FFFF00">32769</span>
             Address     001d.707b.4200
             Cost        19
             Port        25 (FastEthernet1/0/23)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0022.be5a.0680
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa1/0/21         Altn BLK 19        128.23   P2p
Fa1/0/23         Root FWD 19        128.25   P2p
</pre>
Now, I issue `spanning-tree vlan 1 root primary` command on the Access
switch and compare the output of the `show spanning-tree` command:
<pre>
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    <span style="font-weight: bold;background-color: #FFFF00">24577</span>
             Address     0022.be5a.0680
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24577  (priority 24576 sys-id-ext 1)
             Address     0022.be5a.0680
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa1/0/21         Desg FWD 19        128.23   P2p
Fa1/0/23         Desg FWD 19        128.25   P2p
</pre>
Now let's see the verification commands on Distribution bridges while
Access switch is advertising itself as a root brindge.
<pre>
Distribution_1#<b>show spanning-tree inconsistentports</b>

Name                 Interface                Inconsistency
-------------------- ------------------------ ------------------
VLAN0001             FastEthernet1/0/23       Root Inconsistent

Number of inconsistent ports (segments) in the system : 1
</pre>
<pre>
Distribution_2#<b>show spanning-tree inconsistentports</b>

Name                 Interface              Inconsistency
-------------------- ---------------------- ------------------
VLAN0001             FastEthernet1/0/21     Root Inconsistent

Number of inconsistent ports (segments) in the system : 1
</pre>

Now I am going to suppress the superior BPDU by giving a higher priority
to Access.
<pre>
*Mar  1 01:27:42.601: %SPANTREE-2-ROOTGUARD_UNBLOCK: Root guard unblocking port FastEthernet1/0/23 on VLAN0001.
</pre>

## Loop Guard:
If you ever used fiber cables you might have noticed that there is a
different connector to transmit and receive traffic.

If one of the cables (transmit or receive) fails we'll have a
unidirectional link failure and this can cause spanning tree loops.
There are two protocols that can take care of this problem:

* LoopGuard
* UDLD ( Unidirectional Link Detection )

In STP, the blocked ports will remain in the Blocking state as long as
a steady flow of BPDUs is received. If BPDUs are being sent over a link
but the flow of BPDUs stops for some reason, the last-known BPDU is kept
until the Max Age timer expires. Then that BPDU is flushed, and the switch
thinks there is no longer a need to block the port. The switch then
moves the port through the STP states until it begins to forward
traffic and forms a bridging loop.

if a port enabled with loopguard stops hearing BPDUs from the designated
port on the segment, instead of transitioning into forwarding, it goes
into the loop-inconsistent state. When BPDUs are received on the port
again, Loop Guard allows the port to move through the normal STP states
and become active.

### Configuration:
To simulate BPDU transmit, I will use:
```
Switch(config)#interface fastEthernet 1/0/24
Switch(config-if)spanning-tree portfast trunk
Switch(config-if)#spanning-tree bpdufilter enable
```

If you are configuring this in a real network, you generally want it
enabled on any port that is non-designated in your topology.
<pre>
Distribution_2(config-if)#<b>spanning-tree guard loop</b>
Distribution_2#show spanning-tree interface fastEthernet 1/0/24 detail
 Port 26 (FastEthernet1/0/24) of VLAN0001 is forwarding
   Port path cost 19, Port priority 128, Port Identifier 128.26.
   Designated root has priority 32769, address 001d.707b.4200
   Designated bridge has priority 32769, address 001d.707b.4200
   Designated port id is 128.26, designated path cost 0
   Timers: message age 1, forward delay 0, hold 0
   Number of transitions to forwarding state: 2
   Link type is point-to-point by default
   <span style="background-color: #FFFF00">Loop guard is enabled on the port</span>
   BPDU: sent 48, received 2999
</pre>

## Unidirectional Link Detection (UDLD):
The other protocol we can use to deal with unidirectional link failures
is called UDLD (UniDirectional Link Detection). This protocol is not
part of the spanning tree toolkit but it does help us to prevent loops.

Simply said UDLD is a layer 2 protocol that works like a keepalive
mechanism. You send hello messages, you receive them and life is good.
As soon as you still send hello messages but don't receive them anymore
you know something is wrong and we'll block the interface.

* Cisco proprietary
* Unidirectional link usually occurs in fiber optic


To simulate unidirectional link failure is a tricky part here.
LoopGuard was easier because it was based on BPDUs. UDLD runs its own
layer 2 protocol by using the proprietary MAC address 0100.0ccc.cccc.
We can create a filter to block the UDLD traffic:
```
Switch(config)#mac access-list extended UDLD-FILTER
Switch(config-ext-macl)#deny any host 0100.0ccc.cccc
Switch(config-ext-macl)#permit any any
Switch(config-ext-macl)#exit
Switch(config)#interface fa1/0/19
Switch(config-if)#mac access-group UDLD-FILTER in
```
Then a loop occurs.
By default, UDLD is disabled on all switch ports. To enable it globally,
use the following global configuration command:
```
Switch(config)#udld ?
  aggressive  Enable UDLD protocol in aggressive mode on fiber ports except where locally configured
  enable      Enable UDLD protocol on fiber ports except where locally configured
  message     Set UDLD message parameters
```
When you set UDLD to normal it will mark the port as undetermined but
it won't shut the interface when something is wrong. This is only used
to "inform" you but it won't stop loops.

Aggressive is a better solution. When it loses connectivity to a
neighbor it will send a UDLD frame 8 times in a second. If the neighbor
does not respond the interface will be put in err-disable mode.

You also can enable or disable UDLD on individual switch ports, if
needed, using the following interface configuration command:
Here, you can use the disable keyword to completely disable UDLD on a
fiber-optic interface.
```
Switch(config-if)# udld { enable | aggressive | disable }
```
In order for UDLD to work we have to configure on both ends because UDLD
is trying to establish a neighbor relationship with the other end.

To verify:
```
Switch#show udld fastEthernet 1/0/19
```
```
Switch#debug udld events
```
LoopGuard and UDLD both solve the same problem: Unidirectional Link
failures. They have some overlap but there are a number of differences,
here's an overview:

|   | LoopGuard | UDLD |
| --- | --- | --- |
| **Configuration** | Global / per port	 | Global (for fiber) / per port |
| **Per VLAN?**	| Yes | No, per port |
| **Autorecovery** | Yes | Yes – requires err-disable timeout. |
| **Protection against STP failures because of unidirectional link failures** | Yes – need to enable it on all root and alternate ports | Yes – need to enable it on all interfaces |
| **Protection against STP failures because no BPDUs are sent** | Yes | No |
| **Protection against miswiring** | No | Yes |