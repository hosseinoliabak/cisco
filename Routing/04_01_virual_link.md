# Virtual Link Configuration

This is an example we work on

![ospf](https://user-images.githubusercontent.com/31813625/33810819-c3e14df0-ddd7-11e7-8f8f-b369b2487e63.png)

### Basic OSPF Confoguration

**R1:**
<pre>
router ospf 1
 network 12.12.12.0 0.0.0.3 area 12
</pre>

**R2:**
<pre>
router ospf 1
 network 12.12.12.0 0.0.0.3 area 12
 network 172.16.234.0 0.0.0.255 area 234
</pre>

**R3:**
<pre>
router ospf 1
 network 35.35.35.0 0.0.0.7 area 0
 network 172.16.234.0 0.0.0.255 area 234
</pre>

**R4:**
<pre>
router ospf 1
 area 234 virtual-link 2.2.2.2
 network 45.45.45.0 0.0.0.7 area 0
 network 172.16.234.0 0.0.0.255 area 234
</pre>

**R5:**
<pre>
router ospf 1
 network 10.10.0.0 0.0.7.255 area 0
 network 35.35.35.0 0.0.0.7 area 0
 network 45.45.45.0 0.0.0.7 area 0
</pre>

**Basic verification on R5:**
1. Routing Table:
<pre>
R5#<b>show ip route ospf</b>
Gateway of last resort is not set

      172.16.0.0/24 is subnetted, 1 subnets
O IA     172.16.234.0 [110/2] via 45.45.45.4, GigabitEthernet0/4
                      [110/2] via 35.35.35.3, GigabitEthernet0/3
</pre>

2. LSDB:
<pre>
R5#<b>show ip ospf database</b>

            OSPF Router with ID (50.5.5.5) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
3.3.3.3         3.3.3.3         616         0x80000008 0x0094A3 1
4.4.4.4         4.4.4.4         613         0x80000008 0x00B042 1
50.5.5.5        50.5.5.5        612         0x80000004 0x005037 6

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
35.35.35.5      50.5.5.5        615         0x80000001 0x00A2A1
45.45.45.5      50.5.5.5        611         0x80000001 0x006BB6

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
172.16.234.0    3.3.3.3         607         0x80000002 0x0099ED
172.16.234.0    4.4.4.4         602         0x80000004 0x00770A
</pre>

### Virtual Link Configuration
**R2:**
<pre>
R2(config)# <b>router ospf 1</b>
R2(config-router)#<b>area 234 virtual-link 4.4.4.4</b>
R2(config-router)#<b>area 234 virtual-link 3.3.3.3</b></pre>

When R2 Configured, we received these console logs on R3 and R4 every 10 seconds
<pre>
R3#
*Dec  9 02:27:56.261: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 172.16.234.2, GigabitEthernet0/0
R3#
*Dec  9 02:28:05.558: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 172.16.234.2, GigabitEthernet0/0

R4#
*Dec  9 02:29:14.726: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 172.16.234.2, GigabitEthernet0/0
R4#
*Dec  9 02:29:24.691: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 172.16.234.2, GigabitEthernet0/0
</pre>

**R3:**
<pre>
R3(config)#<b>router ospf 1</b>
R3(config-router)#<b>area 234 virtual-link 2.2.2.2</b>
*Dec  9 02:31:02.743: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on OSPF_VL1 from LOADING to FULL, Loading Done
</pre>

* Then we no longer received the console log for R3 but R4 had still in every 10 seconds
* For now let's look at the tables on R5
<pre>
R5#<b>show ip route ospf</b>
Gateway of last resort is not set

      12.0.0.0/30 is subnetted, 1 subnets
<b>O IA     12.12.12.0 [110/3] via 35.35.35.3, GigabitEthernet0/3</b>
      172.16.0.0/24 is subnetted, 1 subnets
O IA     172.16.234.0 [110/2] via 45.45.45.4, GigabitEthernet0/4
                      [110/2] via 35.35.35.3, GigabitEthernet0/3
</pre>

<pre>
R5#<b>show ip ospf database</b>

            OSPF Router with ID (50.5.5.5) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
2.2.2.2         2.2.2.2         2     (DNA) 0x80000007 0x00055F 1
3.3.3.3         3.3.3.3         376         0x8000000B 0x00E986 2
4.4.4.4         4.4.4.4         1109        0x8000000A 0x00AC44 1
50.5.5.5        50.5.5.5        856         0x80000005 0x004E38 6

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
35.35.35.5      50.5.5.5        856         0x80000002 0x00A0A2
45.45.45.5      50.5.5.5        856         0x80000002 0x0069B7

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
12.12.12.0      2.2.2.2         65    (DNA) 0x80000001 0x00937F
172.16.234.0    2.2.2.2         65    (DNA) 0x80000001 0x00B9D2
172.16.234.0    3.3.3.3         719         0x80000003 0x0097EE
172.16.234.0    4.4.4.4         667         0x80000005 0x00750B
</pre>

**R4:**
<pre>
R4(config)#<b>router ospf 1</b>
R4(config-router)#<b>area 234 virtual-link 2.2.2.2</b>
*Dec  9 02:33:37.668: %OSPF-5-ADJCHG: Process 1, Nbr 2.2.2.2 on OSPF_VL0 from LOADING to FULL, Loading Done
</pre>

* Now let's check the routing table and LSDB in R5
<pre>
R5#<b>show ip route ospf</b>
Gateway of last resort is not set

      12.0.0.0/30 is subnetted, 1 subnets
O IA     12.12.12.0 [110/3] via <b>45.45.45.4</b>, GigabitEthernet0/4
                    [110/3] via <b>35.35.35.3</b>, GigabitEthernet0/3
      172.16.0.0/24 is subnetted, 1 subnets
O IA     172.16.234.0 [110/2] via 45.45.45.4, GigabitEthernet0/4
                      [110/2] via 35.35.35.3, GigabitEthernet0/3
</pre>

<pre>
R5#<b>show ip ospf database</b>

            OSPF Router with ID (50.5.5.5) (Process ID 1)

		Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
2.2.2.2         2.2.2.2         2     (DNA) 0x80000003 0x00F8A3 2
3.3.3.3         3.3.3.3         455         0x80000003 0x003048 2
4.4.4.4         4.4.4.4         353         0x80000003 0x00032F 2
50.5.5.5        50.5.5.5        470         0x80000003 0x005236 6

		Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
35.35.35.5      50.5.5.5        470         0x80000001 0x00A2A1
45.45.45.5      50.5.5.5        475         0x80000001 0x006BB6

		Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
12.12.12.0      2.2.2.2         12    (DNA) 0x80000001 0x00937F
172.16.234.0    2.2.2.2         12    (DNA) 0x80000001 0x00B9D2
172.16.234.0    3.3.3.3         507         0x80000001 0x009BEC
172.16.234.0    4.4.4.4         514         0x80000001 0x007D07
</pre>
<pre>
R5#<b>ping 12.12.12.1</b>
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 12.12.12.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/9/11 ms
</pre>

## Authentication
* Customer's requirement:
  1. R2 and R3 authenticate each other using clear text
  2. R2 and R4 authentocate each other using MD5

<pre>
R2(config-router)#<b>area 234 virtual-link 3.3.3.3 authentication-key cisco</b>
R2(config-router)#<b>area 234 virtual-link 4.4.4.4 authentication message-digest-key 89 md5 cisco</pre></b>
<pre>
R3(config-router)#<b>area 234 virtual-link 2.2.2.2 authentication-key cisco</b></pre>
<pre>
R4(config-router)#<b>area 234 virtual-link 2.2.2.2 authentication message-digest-key 89 md5 cisco</b></pre>