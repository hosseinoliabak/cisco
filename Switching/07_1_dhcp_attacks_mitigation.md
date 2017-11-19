# DHCP attacks and mitigation methods
## DHCP spoofing
DHCP spoofing occurs when an attacker attempts to respond to DHCP requests and trying to list themselves (spoofs) as the default gateway or DNS server, hence, initiating a man in the middle attack.

## DHCP Starvation
A DHCP starvation attack works by broadcasting DHCP requests with spoofed MAC addresses.

## DHCP snooping
These DHCP attacks can be mitigated by configuring DHCP Snooping which enables specific ports only to pass DHCP traffic. All other ports will be untrusted and can only send DHCP requests. If a DHCP offer is detected in a untrusted port, it will be shut down.

optionaly, first, because some DHCP servers don't like option 82, we desiable option 82 insertion from global configuration:
<pre>
SW1(config)#<b>no ip dhcp snooping information option<b></pre>


<img src="https://user-images.githubusercontent.com/31813625/32983666-288fa36a-cc66-11e7-8079-b7983f185dd0.png" width="733" height="274" />

When we Apply `ip dhcp snooping vlan X` then all interfaces on VLAN X become untrusted.

If you want the switch to remember the DHCP snooping binding database across the reboots, you should configure DHCP snooping database agent:
<pre>
SW1(config)#<b>ip dhcp snooping database flash:snoop.txt<b></pre>

To mitigate DHCP Starvation, we can limit the packet per second rate by the command below:
<pre>
SW1(config-if)#<b>ip dhcp snooping limit rate ?</b>
  <1-2048>  DHCP snooping rate limit
</pre>

It looks like so simple, but how DHCP snooping achieve this is not such simple:
### DHCP Snooping Binding Database
consists of:
* DHCP Client MAC address
* The VLAN client resides in
* The leased IP address of the client
* The interface where the client is connected

<pre>
SW1#<b>show ip dhcp snooping</b></pre>

To show the leases from our legitimate DHCP server. Very important and necessary information:
<pre>
SW1#<b>show ip dhcp snooping binding</b></pre>

<pre>
SW1#<b>show ip dhcp snooping binding</b></pre>

<pre>
SW1#<b>debug ip dhcp snooping packet</b></pre>
