# Protecting the Spanning Tree Protocol Topology
## BPDU Guard:
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