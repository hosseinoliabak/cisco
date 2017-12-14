# EIGRP Route Summarization
* auto-summary:
  * Advertises the summary routes to all of its neighbors.
  * It is disabled by default as of IOS 15.0(1)M.
  * In older IOSs it was enabled by default, so you needed to issue `no auto-summary` where you decided to disable it.
  * Where to check if this is disabled or enabled? `show ip protocols | begin eigrp`
* Manual summary
  * Manual summarization is configured per interface.
  * I will be going to work on it in this topic
  * It can be done in conjunction with auto summarization or independently
* EIGRP will not send a classful summary route to a neighbor who is advertising a subnet of that major network (Split Horizon)

### Configuration

I took the topology of 05_02 file:

![eigrp_stub](https://user-images.githubusercontent.com/31813625/33972008-85ed1f0a-e049-11e7-93bf-d37ad7d8713d.png)

Let's look at the current R1's EIGRP's route
<pre>
R1#<b>show ip route eigrp</b>
Gateway of last resort is not set

      10.0.0.0/24 is subnetted, 2 subnets
<b>D        10.10.3.0 [90/16000] via 192.168.12.2, 00:30:16, GigabitEthernet0/2
D        10.10.4.0 [90/16000] via 192.168.12.2, 00:30:16, GigabitEthernet0/2</b>
      192.168.23.0/29 is subnetted, 1 subnets
D        192.168.23.0
           [90/15360] via 192.168.12.2, 00:30:16, GigabitEthernet0/2
      192.168.24.0/29 is subnetted, 1 subnets
D        192.168.24.0
           [90/15360] via 192.168.12.2, 00:30:16, GigabitEthernet0/2
</pre>

I am going to summarize 10.10.3.0 and 10.10.4.0 as a 10.10.0.0 255.255.255.248 route on R2 and advertise it to R1
<pre>
R2(config-router-af-interface)#<b>summary-address 10.10.0.0 255.255.248.0</b>
*Dec 14 03:05:52.255: %DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.12.1 (GigabitEthernet0/1) is resync: summary configured
</pre>
Now, let's see the result on R1
<pre>
R1#sho ip route eigrp
Gateway of last resort is not set

      10.0.0.0/21 is subnetted, 1 subnets
<b>D        10.10.0.0 [90/16000] via 192.168.12.2, 00:00:08, GigabitEthernet0/2</b>
      192.168.23.0/29 is subnetted, 1 subnets
D        192.168.23.0
           [90/15360] via 192.168.12.2, 00:31:57, GigabitEthernet0/2
      192.168.24.0/29 is subnetted, 1 subnets
D        192.168.24.0
           [90/15360] via 192.168.12.2, 00:31:57, GigabitEthernet0/2
</pre>

### Verification
<pre>
R2#<b>show ip protocols | begin eigrp</b>
Routing Protocol is "eigrp 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  EIGRP-IPv4 VR(R2) Address-Family Protocol for AS(1)
    Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0
    Metric rib-scale 128
    Metric version 64bit
    Soft SIA disabled
    NSF-aware route hold timer is 240
    Router-ID: 192.168.24.2
    Topology : 0 (base)
      Active Timer: 3 min
      Distance: internal 90 external 170
      Maximum path: 4
      Maximum hopcount 100
      Maximum metric variance 1
      Total Prefix Count: 6
      Total Redist Count: 0

  Automatic Summarization: disabled
  <b>Address Summarization:
    10.10.0.0/21 for Gi0/1</b>
      Summarizing 2 components with metric 1392640
  Maximum path: 4
  Routing for Networks:
    192.168.12.2/32
    192.168.23.2/32
    192.168.24.2/32
  Routing Information Sources:
    Gateway         Distance      Last Update
    192.168.12.1          90      00:05:29
    192.168.24.4          90      00:37:14
    192.168.23.3          90      00:37:14
  Distance: internal 90 external 170
</pre>

One more command I'd like to show:
<pre>
R1#<b>show ip eigrp traffic</b>
EIGRP-IPv4 VR(R1) Address-Family Traffic Statistics for AS(1)
  Hellos sent/received: 1003/958
  <b>Updates sent/received: 16/24
  Queries sent/received: 3/0
  Replies sent/received: 0/3</b>
  Acks sent/received: 22/13
  SIA-Queries sent/received: 0/0
  SIA-Replies sent/received: 0/0
  Hello Process ID: 121
  PDM Process ID: 100
  Socket Queue: 0/10000/1/0 (current/max/highest/drops)
  Input Queue: 0/10000/1/0 (current/max/highest/drops)
</pre>
In a stable EIGRP topology the bolded numbers should remain relatively stagnant. If you see these numbers changing
alot, it means something is probably wrong.