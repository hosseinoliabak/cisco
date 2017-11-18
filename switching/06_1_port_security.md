# Port Security:
<pre>
Switch(config-if)#<b>switchport port-security</b>
Command rejected: FastEthernet3/0/1 is a dynamic port.
Switch(config-if)#<b>switchport mode access</b>
Switch(config-if)#<b>switchport port-security</b>
Switch(config-if)#<b>switchport port-security mac-address ?</b>
  H.H.H   48 bit mac address
  sticky  Configure dynamic secure addresses as sticky
Switch(config-if)#<b>switchport port-security violation ?</b>
  protect   Security violation protect mode
  restrict  Security violation restrict mode
  shutdown  Security violation shutdown mode
</pre>
* **protect:** Drops packets with unknown source addresses
* **restrict:** Drops packets with unknown source addresses + produce a log + increment SecurityViolation counter
* **shutdown:** Puts the interface into the error-disabled state immediately and sends an SNMP trap notification.

<pre>
Switch#<b>show port-security</b>
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
    Fa3/0/1              1            0                  0         Shutdown
---------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 6144
</pre>

<pre>
Switch#<b>show interfaces status err-disabled</b>
</pre>