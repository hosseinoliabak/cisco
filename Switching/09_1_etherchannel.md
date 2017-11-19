# Aggregating switch links with Etherchannel
Make sure all ports participating in Etherchannel have the same configuration
* Duplex
* Speed
* Native VLAN
* Allowed VLANs
* Switchport mode (Access/Trunk)

## PAgP and LACP Modes
* PAgP (Cisco proprietary)
* LACP (A subcomponent of IEEE 802.3ad)
### PAgP
The interface can configured as:
* <b>On:</b> interface becomes member of the etherchannel but does not negotiate
* <b>Desirable:</b> interface will actively ask the other side to become an etherchannel
* <b>Auto:</b> interface will wait passively for the other side to ask to become an etherchannel
* <b>Off:</b> no etherchannel configured on the interface

| | On | Desirable | Auto | Off |
| --- | --- | --- | --- | --- |
| __On__ | Yes | No | No | No |
| __Desirable__ | No | Yes | Yes | No |
| __Auto__ | No | Yes | No | No
| __Off__ | No | No | No | No |