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
      * Routing IGP, BGP, etc.
    * Traffic Encryption
      * IPsec

### How DMVPN Works
* Two main components
  * DMVPN Hub/ NHRP Server
  * DMVPN Spokes / NHRP Clients
* Spoke/Client register with Hub/Server
  * Spokes manually specify Hub's address
  * Hub dynamically learns Spokes' VPN address and NBMA address
  * Sent via NHRP request (NHRP is like Frame Relay or ATM inverse ARP instead of mapping L2 to L3 information, we are now mapping a tunnel IP address to an NBMA address.)
* Spokes establish tunnel to hub
  * Used exchange IGP routing information
  * Spokes know routes to other spokes somehow: could be through static route, IGP, or BGP
  * These routes learned via the tunnel to the hub
* Once the spoke-to-spoke tunnel is formed, hub only used for control plane exchange
* It is possible you have multiple NHRP Servers, multiple levels of tunnels.
* We will be simple design, single DMVPN with a single hub

![dmvpn](https://user-images.githubusercontent.com/31813625/35466811-6590b56a-02d5-11e8-8e4a-692f5e8ccb19.png)

**DMVPN versions (Phases)**
* DMVPN Phase 1
  * All spokes uses regular point-to-point GRE tunnel interfaces
  * No direct spoke-to-spoke communication

* DMVPN Phase 2
  * We have spoke-to-spoke tunneling
  * Spoke routers should have a route to the other spoke
  * Next-hop IP address of the route has to be the remote spoke

* DMVPN Phase 3
  * Spoke router doesn't need to have specific route to the other spoke
  * Doesn't matter what the next-hop IP address is
  * Spoke1 sends the traffic directly to the hub. Hub sees that another spoke is
  destination. Hub then sends an NHRP redirect to both spokes.
  * Then spokes send an NHRP resolution to retrieve the NBMA addresses, then spokes install
a new entry in the routing table