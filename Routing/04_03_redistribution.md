# Redistribution into OSPF

## Redistributing Connected Routes
* Provides and alternative to the network keyword but there are some significant differences:
  * When you use network keyword:
    * OSPF generates type-a and type-2 LSAs
    * Shows up as `O` or `O IA` routes
    * Enables OSPF on the interface
  * When you redistribute connected routes
    * Generates external route (Type-5 or Type-7 LSA)
    * Shows up as `O E1` or `O E2` routes
    * Does not enable OSPF on the interface
### Configuration Example

![ospf_r_con](https://user-images.githubusercontent.com/31813625/33814804-cd7d610a-ddfa-11e7-982f-737215031d51.png)

**R1**
<pre>
R1#show run | s ospf
router ospf 1
 network 10.10.1.1 0.0.0.0 area 0
</pre>
**ASBR**
<pre>
ASBR(config)#<b>router ospf 1</b>
ASBR(config-router)#<b>network 10.10.1.2 0.0.0.0 area 0</b>
ASBR(config-router)#<b>redistribute connected subnets</b></pre>


Verification on R1

<pre>
R1#<b>show ip route ospf</b>
       O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2

Gateway of last resort is not set

      1.0.0.0/8 is variably subnetted, 3 subnets, 3 masks
<b>O <u>E2</u>     1.1.1.0/30 [110/20] via 10.10.1.2, 00:03:06, GigabitEthernet0/0
O <u>E2</u>     1.1.2.0/24 [110/20] via 10.10.1.2, 00:03:06, GigabitEthernet0/0
O <u>E2</u>     1.3.0.0/16 [110/20] via 10.10.1.2, 00:03:06, GigabitEthernet0/0</b></pre>

<pre>
R1#<b>show ip ospf database</b>

            OSPF Router with ID (10.10.1.1) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.3.0.1         1.3.0.1         471         0x80000003 0x0045B2 1
10.10.1.1       10.10.1.1       470         0x80000002 0x00CD0C 1

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.10.1.1       10.10.1.1       470         0x80000001 0x00649D

		Type-5 AS External Link States

Link ID         ADV Router      Age         Seq#       Checksum Tag
1.1.1.0         1.3.0.1         483         0x80000001 0x008A11 0
1.1.2.0         1.3.0.1         483         0x80000001 0x009106 0
1.3.0.0         1.3.0.1         483         0x80000001 0x008F08 0
</pre>

LSID<sub>Type-4</sub> = External Network Number. Let's open one of the AS External LSAs

<pre>
R1#<b>show ip ospf database external 1.1.1.0</b>

            OSPF Router with ID (10.10.1.1) (Process ID 1)

		Type-5 AS External Link States

  LS age: 605
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 1.1.1.0 (External Network Number )
  Advertising Router: 1.3.0.1
  LS Seq Number: 80000001
  Checksum: 0x8A11
  Length: 36
  Network Mask: /30
	Metric Type: 2 (Larger than any link state path)
	MTID: 0
	Metric: 20
	Forward Address: 0.0.0.0
	External Route Tag: 0
</pre>
