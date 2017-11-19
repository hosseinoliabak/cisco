# Dynamic ARP Inspection (DAI)
## ARP Poisioning:
<img src="https://user-images.githubusercontent.com/31813625/32990764-19828666-ccfd-11e7-981b-ef883a3a5801.png" width="723" height="325" />

## DAI
<pre>
SW1(config)#<b>interface vlan 1</b>
SW1(config-if)#<b>ip address 192.168.7.1 255.255.255.0</b>
SW1(config-if)#<b>no shutdown</b> 
SW1#<b>show interfaces vlan 1 | i bia</b>
  Hardware is Ethernet SVI, address is <b>0097.81b6.8001</b> (bia 0097.81b6.8001)
</pre>
<pre>
SW2-Client#<b>ping 192.168.7.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.7.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 4/5/7 ms
SW2-Client#<b>show arp 192.168.7.1</b>
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.7.1             0   <b>0097.81b6.8001</b>  ARPA   Vlan1
</pre>
Now the attacker does ARP Poisoning, let's see agian the ARP table of the client:
<pre>
SW2-Client#<b>show arp 192.168.7.1</b>
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.7.1             0   <b>b826.ebc5.a07e</b>  ARPA   Vlan1
</pre>

### DAI Configuration:
<pre>
SW1(config)#<b>ip arp inspection vlan 1</b></pre>
Then all interfaces in vlan 1 become untrusted. Then:
* DAI intercepts ARP requests and replies on untrusted interfaces.
* Limits the number of incoming ARP packets to 15 per second on an untrusted interface.
* Compares contents with the DHCP snooping binding database.

Trusted interfaces are usually uplinks.
<pre>
SW1(config)#<b>interface gigabitEthernet 3/3</b>  
SW1(config-if)#<b>ip arp inspection trust</b></pre>
Trusted ports:
* DAI does NOT inspect incoming ARP packets on trusted interfaces

**Note:** DAI does NOT inspect outgoing ARP packets on any interface
 
