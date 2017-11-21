# General Network design
## Cisco three-layered hierarchy
### Core Layer (Also called the Backbone)
* What the Core should do/have:
  * High speed Layer 3 transport
  * Low latency for traffic
  * QoS
  * Redundancy/fault tolerance
  * Limit diameter
  * Simplicity
  * Efficiency
* What the Core should NOT do/have:
  * Packet classification for QoS: Voice, Video, Data
  * Inter-VLAN routing: This is for distribution layer
  * Packet filtering/security features
  * Complicated packet manipulation (e.g. PBR)

### Distribution Layer (between Access and Core layers)
* What the Distribution should do/have:
  * Broadcast domain perimeter
  * Inter-VLAN routing
  * WAN connectivity (including Internet)
  * Aggregation of Access Layer switches (e.g., floor switches)
  * QoS/Security mechanisms
  * IP Routing
* What the Distribution should NOT do/have:
  * Direct connectivity to end users
  * Layer 2 switching
  * Extensive STP operation
  * You should not see v oice VLANs

### Access Layer (End-User Connectivity)
* What the Access should do/have:
  * Direct connectivity to end users
  * Layer 2 switching
  * STP operations
  * Voice VLANs
  * PoE
  * Trust classification
  * Port-based security
  * QoS packet classification
  * DAI
* What the Access should NOT do/have:
  * Inter-VLAN routing
  * Direct connectivity to other Access Layer switches
  * Single uplinks to the Distribution Layer
  * Extensive security filtering
  * WAN connectivity

### Some VLAN design notes:
* If your broadcast traffic is more than 25% of your traffic, you should split off into the VLANs
* Don't use VLAN 1 (shutdown! )