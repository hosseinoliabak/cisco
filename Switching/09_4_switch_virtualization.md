# Cisco Switch Virtualization
With STP we have redundant links but we don't have them all in forwarding state. Even we don't have redundancy in Access layer.

<img src="https://user-images.githubusercontent.com/31813625/33027091-36b9b3a6-cde0-11e7-9a44-c3d9c1e601d9.png" width="305" height="350" />


One way of solving this problem is to create a logical switch which is made up of two or more physical switches:

<img src="https://user-images.githubusercontent.com/31813625/33029557-e946ee48-cde6-11e7-9040-7d774d3c6cd0.png" width="305" height="455" />

We can do the same thing in the distribution layer. You can guess the illustration.

<img src="https://user-images.githubusercontent.com/31813625/33031263-c153a35e-cdeb-11e7-8f93-29b095f3a713.png" width="128" height="325" />

The four links between the switch pairs can be combined into a single etherchannel since we have single link between two logical switches. This is called Multi-Chassis Etherchannel.

#### Before and after combining multiple switches
##### Before:
<img src="https://user-images.githubusercontent.com/31813625/33031719-1197c6fa-cded-11e7-87e0-a91a0e2725b0.png" width="683" height="277" />

##### After:
<img src="https://user-images.githubusercontent.com/31813625/33031881-85eb4af4-cded-11e7-8801-243a8a560177.png" width="683" height="277" />

# Cisco Stackwise
Cisco Stackwise allows us to turn multiple physical switches (up to 9 switches) into a single logical switch.
One switch becomes master and others become members. If master fails, another member becomes the master.

### Election process to choose the master
1. User priority
2. Hardware/Software priority (for example IP Service vs IP Base)
3. Default configuration: The switch which has already the configuration
4. Longest uptime:
5. Lowest MAC address:

### Verification
```
SW1#show switch
SW1#show switch stack-ports
SW1#show switch stack-ring speed
SW3#show cdp neighbors
```
SW3 is another switch which is connected to our stack