# Rapid (Per-VLAN) Spanning-Tree (RSTP/RPVST+) - IEEE 802.1w standard
RSTP is not timer-based unlike 802.1D which is timer-based. RSTP uses a
proposal and agreement process to build up the spanning tree faster.

Do you remember the port states in PVST?
* Blocking -> Listening -> Learning -> Forwarding

In RSTP the port states are:
* Discarding -> Learning -> Forwarding

We are still having 2 type os BPDUs:
* Configuration: Version 2 (In PVST it was Version 0)
* TCN: Remains the same as in PVST for compatibility

In PVST the root bridge makes BPDUs and is being flooded by other switches.
In RSTP each switch makes its own BPDU.

### Upgrading to RSTP:
* In a live environment, you may upgrade to RSTP in phases.
* RSTP is backward compatible with legacy STP 802.1D. If a RSTP enabled
port receives a (legacy) 802.1d BPDU, it will automatically configure
itself to behave like a legacy port. It sends and receives 802.1d BPDUs
only.
** You can run 802.1W RSTP with 802.1D STP in the same network, but not
on the same switch (for example per VLANs).

### Link/Port Types:
* **Edge Port:** A port at the "edge" of the network, where only a single
host connects. Traditionally, this has been identified by enabling the
STP PortFast feature. RSTP keeps the PortFast concept for familiarity.
By definition, the port cannot form a loop as it connects to one host,
so it can be placed immediately in the Forwarding state. However, if a
BPDU ever is received on an edge port, the port immediately loses its
edge port status.

To configure a port as an RSTP edge port, use the following interface
configuration command:
<pre>
Switch(config-if)# <b>spanning-tree portfast</b>
</pre>
* **Root port:** The port that has the best cost to the root of the STP
instance. Only one root port can be selected and active at any time,
although alternative paths to the root can exist through other ports.
If alternative paths are detected, those ports are identified as
alternative root ports and immediately can be placed in the Forwarding
state when the existing root port fails.

* **Point-to-point (P2P) port:** Any port that connects to another switch and
becomes a designated port. A quick handshake with the neighboring switch,
rather than a timer expiration, decides the port state. BPDUs are
exchanged back and forth in the form of a proposal and an agreement.
One switch proposes that its port becomes a designated port; if the
other switch agrees, it replies with an agreement message.

* **Shared  port: in the same segment (when a hub is in between)

### Configuration:
Remember that we change the spanning-tree mode in switches in phases.
<pre>
Distribution_1(config)#spanning-tree mode rapid-pvst
</pre>
<pre>
Distribution_2(config)#spanning-tree mode rapid-pvst
</pre>
<pre>
Switch(config)#spanning-tree mode rapid-pvst
</pre>
See the verification command:
<pre>
Access#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
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
Fa1/0/21         Altn <b>BLK<b> 19        128.23   <b>P2p<b> Peer(STP)
Fa1/0/23         Root FWD 19        128.25   P2p
</pre>
`BLK` in RSTP means in discarding state

Now, I am going to disconnect Fa1/0/23 and see the output of
`debug spanning-tree events` command on Access switch:
<pre>
00:15:<b>21</b>: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet1/0/23, changed state to down
00:15:<b>21</b>: RSTP(1): updt roles, root port Fa1/0/23 going down
00:15:<b>21</b>: RSTP(1): Fa1/0/21 is now root port
</pre>
Now, I am going to connect back the port Fa1/0/23:
<pre>
00:15:<b>40</b>: %LINK-3-UPDOWN: Interface FastEthernet1/0/23, changed state to up
00:15:<b>40</b>: RSTP(1): initializing port Fa1/0/23
00:15:<b>40</b>: RSTP(1): Fa1/0/23 is now designated
00:15:<b>40</b>: RSTP(1): transmitting a proposal on Fa1/0/23
00:15:<b>40</b>: RSTP(1): updt roles, received superior bpdu on Fa1/0/23
00:15:<b>40</b>: RSTP(1): Fa1/0/23 is now root port
00:15:<b>40</b>: RSTP(1): Fa1/0/21 blocked by re-root
00:15:<b>40</b>: RSTP(1): synced Fa1/0/23
00:15:<b>40</b>: RSTP(1): Fa1/0/21 is now alternate
00:15:<b>40</b>: RSTP(1): transmitting an agreement on Fa1/0/23 as a response to a proposal
</pre>





