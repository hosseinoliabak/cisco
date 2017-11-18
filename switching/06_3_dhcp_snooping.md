# DHCP Attacks
## DHCP spoofing:
DHCP spoofing occurs when an attacker attempts to respond to DHCP requests and trying to list themselves (spoofs) as the default gateway or DNS server, hence, initiating a man in the middle attack.

This can be mitigated by configuring DHCP Snooping which enables specific ports only to pass DHCP traffic. All other ports will be untrusted and can only send DHCP requests. If a DHCP offer is detected in a untrusted port, it will be shut down.

<img src="https://user-images.githubusercontent.com/31813625/32983666-288fa36a-cc66-11e7-8079-b7983f185dd0.png" width="607" height="212" />

When we Apply `ip dhcp snooping vlan X` then all interfaces on VLAN X become untrusted.

## DHCP Starvation:
A DHCP starvation attack works by broadcasting DHCP requests with spoofed MAC addresses.

To mitigate DHCP Starvation, we can limit the packet per second rate by the command below:
<pre>
SW1(config-if)#<b>ip dhcp snooping limit rate ?</b>
  <1-2048>  DHCP snooping rate limit
</pre>
