# Firewall
* Firewalls create zones and the zones give me the isolation.
Then I can control the traffic between my zones
* The firewall that Cisco sells it Adaptive Cisco Appliance (ASA) AKA: NGFW
  * For ASAs we have physical and virtual appliances
* Cisco also sells Virtual Security Gateway (VSG) firewalls
* A router could also act as a firewall using Context-Based Access Control (CBAC)
 or Zone-Based Policy Firewall (ZBPFW) or abbreviated to ZBFW

#### Firewall Implementation
* Packet filtering on routers and switches (ACL)
* Dedicated firewall appliances (e.g ASA)
* Complex systems integrating multiple hosts and appliances

#### Requirements of all firewall implementations
* Must be resistant to attacks
* Must be the only transit point between network zones
  * Inside Zone: Trusted (meaning that the Security level is 100)
  * Outside Zone: Security level = 0
  * DMZ: Public facing services in our organization
* Traffic form higher to lower security level is allowed to pass.
It will create an entry in the state table and the reply is allowed to come back.
On the other hand, traffic from a lower to a higher security level interface
is gonna be denied. If we want to allow some traffic for example traffic on
port 80 to be allowed from a lower security level to a higher security level, we have
to permit that traffic by an ACL
* If security levels are the same, by default that traffic is not allowed. But
we can allow the traffic using command below:
<pre>
ciscoasa(config)# <b>same-security-traffic permit inter-interface</b>
</pre>
* Configuration based on organization's security policy
* Should be components in larger security architecture and solution?

### Packet filtering firewalls (Stateless firewalls)
* Each packet that passes through a firewall is going to be filtered independent of one another.
* Permit/Deny-based on just the information in the header
* Implemented by ACLs
* Packet filtering is very fast

### Stateful firewalls
* Control sessions
* Information is stored in a state table (in ASA it is called a Connection Table)
* Stateful session flow table contains: Source and destination IP addresses,
Source and Destination Port numbers, TCP sequencing information,
and additional flag for each TCP or UDP connection associated with a particular session
* Decisions are made based on information in the state table
* Usually when a traffic is allowed to leave, then we want to receive the return traffic
* Still Implement ACLs
  * if the traffix is allowed, then an entry will be created in the state table
  * if the traffic is not allowed, the traffic is dropped and we don't need to be worry about tracking the traffic
* Cannot detect Layer 7 attacks

### Application firewalls (Proxy servers)
* Servers sees a connection from the proxy, not the client
* Proxy server needs specialized coding on a per-application basis
* Proxy servers should be transparent to the end users, the client application
may need to have special coding as well
* Slower than Stateful and Stateless Firewalls
* We can combine the proxy server with stateful firewall
  * This eliminate the need to worry about other protocols (other than the web traffics)
  that are traversing the network

### Next Generation Firewalls
* They offer all statefull, stateless, and application firewall features and much mode such as:
* URL filtering
* Application Visibility and Control (AVC): All the OSI protocol stack layers
* Context Awareness: Who? What kind of device? Where they are connected (Switch, AP, ...)? How?
* IPS (deep packet inspection)
* Advanced Malware Protection (Cisco AMP project)
* Identity-based Access
* Cisco Deiscovery Agent (CDA) talks using WMI to Active Directory

### Logging
Firewalls are so chatty, meaning that they prodoce a lot of logs. It is easy
to become overwhelmed with the log volume. You don't want to miss important
firewall log messages
* Implement a 3rd party tool such as Splunk to help with logging

<pre>
ciscoasa(config)# <b>show logging</b>
Syslog logging: disabled
    Facility: 20
    Timestamp logging: disabled
    Hide Username logging: enabled
    Standby logging: disabled
    Debug-trace logging: disabled
    Console logging: disabled
    Monitor logging: disabled
    Buffer logging: disabled
    Trap logging: disabled
    Permit-hostdown logging: disabled
    History logging: disabled
    Device ID: disabled
    Mail logging: disabled
    ASDM logging: disabled
</pre>
<pre>
ciscoasa(config)# <b>logging on</b></pre>
<pre>
ciscoasa(config)# <b>logging trap ?</b>

configure mode commands/options:
  <0-7>          Enter syslog level (0 - 7)
  WORD           Specify the name of logging list
  alerts         Immediate action needed           (severity=1)
  critical       Critical conditions               (severity=2)
  debugging      Debugging messages                (severity=7)
  emergencies    System is unusable                (severity=0)
  errors         Error conditions                  (severity=3)
  informational  Informational messages            (severity=6)
  notifications  Normal but significant conditions (severity=5)
  warnings       Warning conditions                (severity=4)
</pre>

Note that you cannot do below configuration unless you configured an interface
ahead of time. (how to configure the interface is explained in "asic Confoguration and verification" section)
<pre>
ciscoasa(config)# <b>logging host inside 172.16.0.50</b></pre>
`172.16.0.50` is the host we want to send the logs to

### Modes of Deployment
* Single Routed Mode (default): Act as in L3
* Single Transparent Mode: Act as in L2
* Multiple Context Routed Mode
* Multiple Context Transparent Mode

### High Availability, Failover, and clustering
* Active/Standby Failover
* Active/Active Failover
* Clustering: The ASAs are connected to switch ports. We configure the link between
switch ports and ASAs as a big trunk etherchannel
  * For clusterign you need to have a physical device (not virtual. so ASAv doesn't work)

### Overview of the Cisco ASA CLI Command Set
* Five CLI modes similar to Cisco IOS devices:
  * ROM monitor
  * User EXEC
  * Privilege
  * Global Configuration
  * Specific Configuration

### Basic Confoguration and verification
I am using ASAv inside GNS3 which is installed on Ubuntu-based Linux OS

Interface configuration
<pre>
ciscoasa(config)# <b>interface gigabitEthernet 0/0</b>
ciscoasa(config-if)# <b>ip address 10.10.10.1 255.255.255.252</b>
ciscoasa(config-if)# <b>nameif inside</b>
INFO: Security level for "inside" set to 100 by default.
ciscoasa(config-if)# <b>no shutdown</b>
</pre>

<pre>
ciscoasa(config)# <b>show nameif</b>
Interface                Name                     Security
GigabitEthernet0/0       inside                   100
GigabitEthernet0/1       outside                    0
</pre>

**ASDM** is a java-based GUI tool that facilitates the setup, configuration,
monitoring, and troubleshooting of ASAs.

I used David Bombal's great tutorial in link below to setup the ASDM.

https://www.youtube.com/watch?v=QIuZdAcLXs0&feature=youtu.be

First we have to enable HTTPS server on the device:
<pre>
ciscoasa(config)# <b>http server enable</b>
ciscoasa(config)# <b>http 0 0 outside</b>
</pre>

<img src="https://user-images.githubusercontent.com/31813625/33412470-8220ae32-d559-11e7-84f5-aaa09a037cd8.png" />


To preview commands before sending them to the device:

![image](https://user-images.githubusercontent.com/31813625/33460479-4b0d61b2-d5fc-11e7-869f-1603e6c0aa77.png)

 
