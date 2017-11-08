# Configuring a range of interfaces
We knew how to configure a range of interfaces:
```
SW1(config)#interface range gigabitEthernet 3/0 - 3 , gigabitEthernet 2/0 - 2
SW1(config-if-range)#
```

If we want to choose a name for a range of interfaces (Ad-Hoc range):
```
SW1(config)#define interface-range MY_PORTS gig 1/0 - 3, gig 3/0 - 3
SW1(config)#interface range macro MACRO1
SW1(config-if-range)#
```

If we want to define our macro
```
SW1(config)#macro name TEST
Enter macro commands one per line. End with the character '@'.
switchport
switchport mode access
switchport access vlan 10
@
```

Now How to apply the macro TEST to the interface(s):
```
SW1(config)#interface range gig 2/0 - 3
SW1(config-if-range)#macro apply TEST
```
To see the Macros:
```
SW1#show parser macro brief 
    customizable     : TEST
```
```
SW1#show parser macro name TEST
Macro name : TEST
Macro type : customizable
switchport
switchport mode access
switchport access vlan 10
```
We have some predefined macros. We can apply them in global configuration mode:
```
SW1(config)#macro global apply cisco_global
```