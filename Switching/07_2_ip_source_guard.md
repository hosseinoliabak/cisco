# IP Source Guard
DHCP Snooping and IP source guard often configure together because they complement each other really really well.
* Both protect the network at layer 3.
* DHCP Snooping Prevents unauthorized DHCP servers from handing out IP addresses.
* IP Source Guards prevents device from using an IP address it's not entitled to.
What? mmmmm! Does it sound like an ACL? That is true, it effectively behaves like an ACL, but the configuration is very different.

### IP Source Binding Table:
IP Source Binding Table is generated using the data from DHCP snooping database as well as any manually configured IP source bindings.
This means that has an entry in the DHCP snooping binding database is allowed on the network by IP source guard.

Because the IP source binding table is built from <i>DHCP binding database</i>, you must enable DHCP snooping on the VLANs corresponding to the interfaces you want IP source guard to protect.

<pre>
Switch(config)#<b>interface rang gigabitEthernet 0/0 - 3</b>
Switch(config-if-range)#<b>ip verify source</b>
</pre>


### Configure a manual IP source binding:
Static binding:
<pre>
Switch(config)#<b>ip source binding 0012.3456.789a vlan 1 192.168.115.14 interface gigabitEthernet 3/2</b></pre>

### Verification:
To look at the IP source guard configuration:
<pre>
Switch#<b>show ip verify source</b> 
Interface  Filter-type  Filter-mode  IP-address       Mac-address        Vlan
---------  -----------  -----------  ---------------  -----------------  ----
Gi0/0      ip           inactive-no-snooping-vlan
Gi0/1      ip           inactive-no-snooping-vlan
Gi0/2      ip           inactive-no-snooping-vlan
Gi0/3      ip           inactive-no-snooping-vlan
</pre>
The output would be very messy we can alternatively filter by vlan exclusion.
<pre>
Switch#<b>show ip verify source | e vlan</b></pre>
We don't need to worry about our DHCP servers traffic are getting blocked by IP source guard. IP source guard is smart enough to disable itself on the trusted ports.

To view IP source binding table:
<pre>
Switch#<b>show ip source binding</b> 
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
00:12:34:56:78:9A   192.168.115.14   infinite    static          1     GigabitEthernet3/2
Total number of bindings: 1
</pre>

### Alert:
You can't configure IP source guard on a layer 3 routed interface.