# First-hop redundancy protocols (FHRP) - RFC 2281
* Proper functioning of the FHRP depends on proper functioning of layer 2
* I'll be configuring them on switches but we can use routers as well
## LAB Setup
### Topology:
<img src="https://user-images.githubusercontent.com/31813625/33150615-3f433a56-cfa2-11e7-9292-9fa6a1ef7e9c.png" width="481" height="361" />

### Portmapping:
<pre>
from SW1 Gi1/2 to SW2 Gi1/2
from SW1 Gi2/1 to SW2 Gi2/1
from SW1 Gi1/3 to SW3 Gi1/3
from SW1 Gi3/1 to SW3 Gi3/1
from SW1 Gi0/1 to SW4 Gi0/1
from SW1 Gi1/0 to SW4 Gi1/0
from SW2 Gi2/3 to SW3 Gi2/3
from SW2 Gi0/2 to SW4 Gi0/2
from SW2 Gi2/0 to SW4 Gi2/0
from SW3 Gi3/2 to SW2 Gi3/2
from SW3 Gi0/3 to SW4 Gi0/3
from SW3 Gi3/0 to SW4 Gi3/0
</pre>

## Hot Standby Router Protocol (HSRP)

<img src="https://user-images.githubusercontent.com/31813625/33154294-673ae5dc-cfb5-11e7-9b07-1ea68074138d.png" width="652" height="535" />

### Configuration
* Configure SW1 and SW2 according to the table below
* Enable IP routing on both switches

| | VLAN | SVI |
| --- | --- | --- |
| SW1 | 10 | 172.31.10.1/24 |
| SW1 | 20 | 172.31.20.1/24 |
| SW2 | 10 | 172.31.10.2/24 |
| SW2 | 20 | 172.31.20.2/24 |

* All HSRP Configurations are under the SVI configuration mode
  * Make sure SW1 is always the active router for VLAN 10
  * Make sure SW2 is always the passive router for VLAN 10
  * Use the virtual IP address 172.31.10.254
<pre>
SW1(config)#<b>interface vlan 10</b>
SW1(config-if)#<b>standby 10 ip 172.31.10.254</b>
SW1(config-if)#<b>standby 10 priority 110</b>
SW1(config-if)#<b>standby 10 preempt</b></pre>
<pre>
SW2(config)#<b>interface vlan 10</b>
SW2(config-if)#<b>standby 10 ip 172.31.10.254</b></pre>
<pre>
SW1#<b>show standby</b>
Vlan10 - Group 10
  State is Active
    2 state changes, last state change 00:31:07
  Virtual IP address is 172.31.10.254
  Active virtual MAC address is 0000.0c07.ac0a (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.432 secs
  Preemption enabled
  Active router is local
  Standby router is 172.31.10.2, priority 100 (expires in 10.784 sec)
  Priority 110 (configured 110)
  Group name is "hsrp-Vl10-10" (default)
</pre>
* What if you want to trigger the failover based on something less catastrophic other than switch failure.
  * If either Gi1/2 or Gi2/1 on SW1 goes down, ensure that SW2 becomes the active router for VLAN 10.
<pre>
SW1(config)#<b>int vlan 10</b>
SW1(config-if)#<b>standby 10 track 1 decrement 11</b>
SW1(config-if)#<b>standby 10 track 2 decrement 11</b> 
SW1(config)#<b>track 1 interface gigabitEthernet 1/2 line-protocol</b>
SW1(config-track)#<b>exit</b>
SW1(config)#<b>track 2 interface gigabitEthernet 2/1 line-protocol</b>
SW1(config-track)#<b>end</b></pre>
<pre>
SW1#<b>show track</b>
Track 1
  Interface GigabitEthernet1/2 line-protocol
  Line protocol is Up
    1 change, last change 00:04:51
  Tracked by:
    HSRP Vlan10 10
Track 2
  Interface GigabitEthernet2/1 line-protocol
  Line protocol is Up
    1 change, last change 00:02:20
  Tracked by:
    HSRP Vlan10 10
</pre>
<pre>
SW1#<b>show standby</b> 
Vlan10 - Group 10
  State is Active
    2 state changes, last state change 00:52:17
  Virtual IP address is 172.31.10.254
  Active virtual MAC address is 0000.0c07.ac0a (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.920 secs
  Preemption enabled
  Active router is local
  Standby router is 172.31.10.2, priority 100 (expires in 9.536 sec)
  Priority 110 (configured 110)
    <b>Track object 1 state Up decrement 11</b>
    <b>Track object 2 state Up decrement 11</b>
  Group name is "hsrp-Vl10-10" (default)
</pre>

<pre>
SW1(config)#<b>interface gigabitEthernet 1/2</b>
SW1(config-if)#<b>shutdown</b>
SW1(config-if)#
*Nov 23 00:12:28.842: %TRACK-6-STATE: 1 interface Gi1/2 line-protocol Up -> Down
*Nov 23 00:12:30.813: %LINK-5-CHANGED: Interface GigabitEthernet1/2, changed state to administratively down
*Nov 23 00:12:31.813: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/2, changed state to down
</pre>
Nothing happened. Now we have to configure SW2 so that it preempts the SW1
<pre>
SW2(config)#<b>interface vlan 10</b>
SW2(config-if)#<b>standby 10 preempt</b> 
*Nov 23 00:12:36.683: %HSRP-5-STATECHANGE: Vlan10 Grp 10 state Standby -> Active
</pre>
<pre>
SW1#
*Nov 23 00:14:35.206: %HSRP-5-STATECHANGE: Vlan10 Grp 10 state Active -> Speak
*Nov 23 00:14:46.800: %HSRP-5-STATECHANGE: Vlan10 Grp 10 state Speak -> Standby
</pre>

<pre>
SW1#<b>show standby</b> 
Vlan10 - Group 10
  <b>State is Standby</b>
    4 state changes, last state change 00:02:09
  Virtual IP address is 172.31.10.254
  Active virtual MAC address is 0000.0c07.ac0a (MAC Not In Use)
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.448 secs
  Preemption enabled
  Active router is 172.31.10.2, priority 100 (expires in 7.568 sec)
  Standby router is local
  <b>Priority 99 (configured 110)</b>
    <b>Track object 1 state Down decrement 11</b>
    <b>Track object 2 state Up decrement 11</b>
  Group name is "hsrp-Vl10-10" (default)
</pre>


### HSRP Timers
* HSRP routers send Hello to 224.0.0.2 UDP 1985 every 3 seconds. Hold time is 10 seconds.
* HSRP routers are listening to multicast address 224.0.0.2
<pre>
SW1#<b>show ip interface vlan 10  | i Multicast</b>
  Multicast reserved groups joined: <b>224.0.0.2</b>
</pre>
* The timers configured on an active router always override any other timer settings

  <pre>
  SW1(config-if)#<b>standby 10 timers 2 7</b></pre>
<pre>
SW1#<b>show standby vlan 10</b>             
Vlan10 - Group 10
  <b>State is Standby</b>
    1 state change, last state change 01:29:12
  Virtual IP address is 172.31.10.254
  Active virtual MAC address is 0000.0c07.ac0a (MAC Not In Use)
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  <b>Hello time 3 sec (cfgd 2 sec), hold time 10 sec (cfgd 7 sec)</b>
    Next hello sent in 0.640 secs
  Preemption enabled
  Active router is 172.31.10.2, priority 100 (expires in 8.416 sec)
  Standby router is local
  Priority 99 (configured 110)
    Track object 1 state Down decrement 11
    Track object 2 state Up decrement 11
  Group name is "hsrp-Vl10-10" (default)
</pre>

<pre>
SW1(config)#<b>interface gigabitEthernet 1/2</b>
SW1(config-if)#<b>no shutdown</b>
SW1#<b>show standby vlan 10</b>
Vlan10 - Group 10
  <b>State is Active</b>
    2 state changes, last state change 00:01:34
  Virtual IP address is 172.31.10.254
  Active virtual MAC address is 0000.0c07.ac0a (MAC In Use)
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  <b>Hello time 2 sec, hold time 7 sec</b>
    Next hello sent in 0.064 secs
  Preemption enabled
  Active router is local
  Standby router is 172.31.10.2, priority 100 (expires in 5.664 sec)
  Priority 110 (configured 110)
    Track object 1 state Up decrement 11
    Track object 2 state Up decrement 11
  Group name is "hsrp-Vl10-10" (default)
</pre>

### HSRP Authentication
* To prevent unexpected dvices from spoofing or participating in HSRP
* All routers in the same standby group must have an identical authentication method and key
#### Plain-Text HSRP Authentication
* Up to 8 characters
<pre>
SW1(config-if)#<b>standby 20 authentication cisco</b></pre>

#### MD5 Authentication
* Up to 64 characters

##### MD5 Authentication using a key-string 
<pre>
SW1(config-if)#<b>standby 20 authentication md5 key-string cisco</b></pre>

##### MD5 Authentication using key-chain
<pre>
SW1(config)#<b>key chain CHAIN-HSRP-20</b>
SW1(config-keychain)#<b>key 51</b>
SW1(config-keychain-key)#<b>key-string cisco</b>
SW1(config)#<b>interface vlan 20</b>
SW1(config-if)#<b>standby 20 authentication md5 key-chain CHAIN-HSRP-20</b></pre>

* **Note taht**
  * HSRP and STP do NOT have any negotiation with each other, so to increase the performance, you have to design the HSTP Active to be STP root bridge as well
  * HSRP version 1 is the default version
  * the identifier is not the HSRP group; it is the Virtual IP address
  * If the priorities are the same, the switch with higher SVI IP becomes active
  * HSRP has 5 states: Initial, listen, speak, standby and active.
    * **Initial:** This is the beginning state. It indicates HSRP is not running. It happens when the configuration changes or the interface is first turned on
    * **Learn:** The router has not determined the virtual IP address and has not yet seen an authenticated hello message from the active router. In this state, the router still waits to hear from the active router
    * **Listen:** The router knows both IP and MAC address of the virtual router but it is not the active or standby router. For example, if there are 3 routers in HSRP group, the router which is not in active or standby state will remain in listen state.
    * **Speak:** The router sends periodic HSRP hellos and participates in the election of the active or standby router.
    * **Standby:** In this state, the router monitors hellos from the active router and it will take the active state when the current active router fails (no packets heard from active router)
    * **Active:** The router forwards packets that are sent to the HSRP group. The router also sends periodic hello messages
### HSRP Version 2
  * Supports 4096 group numbers
  * Virtual MAC address of 0000.0C9F.FXXX (XXX: HSRP group in hexadecimal)
  * Hello packets are sent to multicast address 224.0.0.102
  * Is configured by`(config-if)# standby version 2 
`
    
## Virtual Router Redundancy Protocol (VRRP)
* VRRP i an open standard protocol defined in RFC 3768
* Virtual MAC: 0000.5e00.01xx (xx is the group number isn hexadecimal format)
* Master router:
  * Equivalent to the HSRP Active router
  * Listens to the virtual MAC and IP address
* Backup router
  * Equivalent to the HSRP Standby router
* In VRRP -unlike the HSRP - the virtal IP address can be the same as an interface IP
  * Allows new VRRP implementation without changing default gateways of the computers
  * The switch whose interface IP matches the virtual IP will always be the master unless is down
* VRRP group number: [1-255] (in HSRP: [0-255] in HSRP, 0 is the default group number)
* Router priority is between and including [0-255] (in HSRP was [0-255])
  * Note that the default VRRP priority number of Master if chosen by IP address is 255
  * default VRRP priority number if backups are 100
  * We can NOT configure a router with priority 0. Priority 0 happens when we decrement to 0 by tracking objects
* Hellos are being sent in a 1-second interval
* hold time is 3 seconds
* Mutlicast address: 224.0.0.18 IP protocol number 112
* In VRRP the preeption is enabled by default

### Configuration
1. Create VLAN 34 on SW3 and SW4
2. Assign IP addresses as below:
  * SW3: 172.31.34.3/24
  * SW4: 172.31.34.4/24
3. SW4 should br the Master whenever possible  
<pre>
SW3(config)#<b>int vl 34</b>
SW3(config-if)#<b>vrrp 34 ip 172.31.34.4</b>
SW4(config)#<b>int vl 34</b>
SW4(config-if)#<b>vrrp 34 ip 172.31.34.4</b>
*Nov 24 15:22:39.230: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Init -> Master
*Nov 24 15:22:39.262: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Init -> Master
</pre>
<pre>
SW4#<b>show vrrp</b>
Vlan34 - Group 34 
  <b>State is Master</b>  
  Virtual IP address is 172.31.34.4
  Virtual MAC address is 0000.5e00.0122
  Advertisement interval is 1.000 sec
  <b>Preemption enabled</b>
  <b>Priority is 255</b> 
  Master Router is 172.31.34.4 (local), priority is 255 
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.003 sec
</pre>

4. Ensure that SW3 takes over as the Master if Gi0/3 on SW4 goes down
<pre>
SW4(config)#<b>track 34 interface gigabitEthernet 0/3 line-protocol</b> 
SW4(config-track)#<b>exit</b>
SW4(config)#<b>interface vlan 34</b>
SW4(config-if)#<b>vrrp 34 track 34 decrement 156</b>
VRRP: Tracking not supported on IP Address owner
SW4(config-if)#<b>ip address 172.31.34.44 255.255.255.0</b>
*Nov 24 17:52:36.218: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Master -> Disable
*Nov 24 17:52:36.219: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Init -> Backup
*Nov 24 17:52:36.224: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Backup -> Disable
*Nov 24 17:52:36.225: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Init -> Backup
*Nov 24 17:52:39.834: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Backup -> Master
SW4(config-if)#<b>vrrp 34 track 34 decrement 1</b>                       
</pre>
<pre>
SW4#<b>show vrrp</b> 
Vlan34 - Group 34 
  <b>State is Master</b>  
  Virtual IP address is 172.31.34.4
  Virtual MAC address is 0000.5e00.0122
  Advertisement interval is 1.000 sec
  Preemption enabled
  <b>Priority is 100</b> 
    Track object 34 state Up decrement 1
  Master Router is 172.31.34.44 (local), priority is 100 
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.609 sec
</pre>
<pre>
SW4(config)#<b>interface gigabitEthernet 0/3</b>
SW4(config-if)#<b>shutdown</b>
SW4(config-if)#
*Nov 24 17:56:36.660: %TRACK-6-STATE: 34 interface Gi0/3 line-protocol Up -> Down
*Nov 24 17:56:38.624: %LINK-5-CHANGED: Interface GigabitEthernet0/3, changed state to administratively down
*Nov 24 17:56:39.624: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/3, changed state to down
*Nov 24 17:56:40.124: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Master -> <b>Backup</b>
</pre>
<pre>
SW4#<b>show vrrp</b> 
Vlan34 - Group 34 
  <b>State is Backup</b>  
  Virtual IP address is 172.31.34.4
  Virtual MAC address is 0000.5e00.0122
  Advertisement interval is 1.000 sec
  Preemption enabled
  <b>Priority is 99</b>  
    Track object 34 state Down decrement 1
  Master Router is 172.31.34.3, priority is 100 
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.609 sec (expires in 3.421 sec)
</pre>
We can see SW3 preempts SW4 when the interface comes back up.
<pre>
SW4(config)#<b>interface gigabitEthernet 0/3</b>
SW4(config-if)#<b>no shutdown</b>
SW4(config-if)#
*Nov 24 18:01:57.961: %LINK-3-UPDOWN: Interface GigabitEthernet0/3, changed state to up
*Nov 24 18:01:57.963: %TRACK-6-STATE: 34 interface Gi0/3 line-protocol Down -> Up
*Nov 24 18:01:58.961: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/3, changed state to up
*Nov 24 18:02:01.373: %VRRP-6-STATECHANGE: Vl34 Grp 34 state Backup -> <b>Master</b>
</pre>

## Gateway Load Balancing Protocol (GLBP)