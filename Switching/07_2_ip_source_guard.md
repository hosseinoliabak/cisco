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



