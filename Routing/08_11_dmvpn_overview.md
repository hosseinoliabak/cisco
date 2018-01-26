# Dynamic Multipoint VPN (DMVPN)
* A point-to-multipoint overlay VPN tunneling technology
* Dynamic and scalable way to build GRE over IPsec site-to-site tunnels
* Spoke to spoke traffic routes over dynamic on-demand tunnels
* Simple configuration management compared to MPLS L2 and L3 VPNs
* Provisioning of new sites will be very straight forward and easy
* Supports transport of multiple protocols
  * IPv4 & IPv6, unicast & multicast, static & dynamic routing
* Uses any IP transport:
  * Any Internet connectivity: T1, DSL, Cable, Ethernet, etc.
  * Support arbitrary number of ISPs
  * Supports going through NAT
* Scalable encryption
  * Spoke-to-spoke tunnels only form as needed
* MPLS L3VPN vs DMVPN
  * MPLS L3 VPN Adds additional routing complexity
  * You have to pay to ISP to provide support of IPv6 in MPLS L3 VPN
  * Inter-provider MPLS L3VPN is difficult
  * In MPLS L3VPN, we have separate circuits for VPN traffic and for Internet access
* MPLS L2VPN with VPLS vs DMVPN
  * Limited connectivity options for example Ethernet only solution
  * Inter-provider MPLS L2VPN is difficult
  * Separate circuit needed for Internet access
* DMVPN Components
  * Can be broken down into two major components
    * Traffic routing
      * mGRE
      * Next Hop Resolution Protocol (NHRP)
    * Traffic Encryption
      * IPsec

### How DMVPN Works
* Two main components
  * DMVPN Hub/ NHRP Server
  * DMVPN Spokes / NHRP Clients
* Spoke/Client register with Hub/Server
  * Spokes manually specify Hub's address
  * Hub dynamically learns Spokes' VPN address and NBMA address
* Spokes establish tunnel to hub
  * Used exchange IGP routing information    

    