# Injecting Default Routes
1. Using `default-information originate`
  * `default-information originate`. We also need to have default route in the ASBR's routing table
  * `default-information originate always`. No need to default route in the ASBR's routing table
2. Using Stub Areas

## default-information originate
The OSPF subcommand default-information originate tells OSPF to create a Type 5 LSA
(used for external routes) for a default route—0.0.0.0/0—and flood it like any other Type 5 LSA.
In other words, it tells the router to create and flood information about a default
route throughout the OSPF domain.

### Configuration

![image](https://user-images.githubusercontent.com/31813625/33861276-02b0fae8-deab-11e7-927b-14a142b63e01.png)

B1's routing table before applying `default-information originate`:
<pre>
B1#<b>show ip route ospf</b>
Gateway of last resort is not set

      10.0.0.0/16 is subnetted, 1 subnets
O IA     10.10.0.0 [110/2] via 172.16.1.3, 00:12:52, GigabitEthernet0/0
                   [110/2] via 172.16.1.2, 00:13:08, GigabitEthernet0/0
</pre>
Let's turn LSA generation debugging on.
<pre>
B1#<b>debug ip ospf lsa-generation</b>
OSPF LSA generation debugging is on
</pre>
Now, I am going to inject the default route to the network o n ASBR1
<pre>
ASBR1(config)#<b>ip route 0.0.0.0 0.0.0.0 209.165.202.130</b>
ASBR1(config-router)#<b>default-information originate</b></pre>
Now le's see what happens on B1
<pre>
B1#<b>show ip route ospf</b>
Gateway of last resort is 172.16.1.3 to network 0.0.0.0

<b>O*E2  0.0.0.0/0 [110/1] via 172.16.1.3, 00:00:03, GigabitEthernet0/0
                [110/1] via 172.16.1.2, 00:00:03, GigabitEthernet0/0</b>
      10.0.0.0/16 is subnetted, 1 subnets
O IA     10.10.0.0 [110/2] via 172.16.1.3, 00:01:48, GigabitEthernet0/0
                   [110/2] via 172.16.1.2, 00:01:48, GigabitEthernet0/0
</pre>

<pre>
B1#<b>show ip ospf database external 0.0.0.0</b>

            OSPF Router with ID (1.1.1.1) (Process ID 1)

		Type-5 AS External Link States

  LS age: 104
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 0.0.0.0 (External Network Number )
  <b>Advertising Router: 99.99.99.99</b>
  LS Seq Number: 80000001
  Checksum: 0x958F
  Length: 36
  Network Mask: /0
	Metric Type: 2 (Larger than any link state path)
	MTID: 0
	Metric: 1
	Forward Address: 0.0.0.0
	External Route Tag: 1
</pre>
Now I am going to configure ASBR2 as well
<pre>
ASBR2(config-router)#<b>default-information originate always</b></pre>
Let's see the result on B1
<pre>
B1#<b>show ip ospf database external 0.0.0.0</b>

            OSPF Router with ID (1.1.1.1) (Process ID 1)

		Type-5 AS External Link States

  LS age: 25
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 0.0.0.0 (External Network Number )
  <b>Advertising Router: 99.99.99.98</b>
  LS Seq Number: 80000001
  Checksum: 0x9B8A
  Length: 36
  Network Mask: /0
	Metric Type: 2 (Larger than any link state path)
	MTID: 0
	Metric: 1
	Forward Address: 0.0.0.0
	External Route Tag: 1

  LS age: 160
  Options: (No TOS-capability, DC, Upward)
  LS Type: AS External Link
  Link State ID: 0.0.0.0 (External Network Number )
  <b>Advertising Router: 99.99.99.99</b>
  LS Seq Number: 80000001
  Checksum: 0x958F
  Length: 36
  Network Mask: /0
	Metric Type: 2 (Larger than any link state path)
	MTID: 0
	Metric: 1
	Forward Address: 0.0.0.0
	External Route Tag: 1
</pre>