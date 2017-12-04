# Remote Access VPN
There are 2 ways of establishing Remote Access VPN:
* Using IPsec (Layer 3): Legacy way
  * IPsec-client should be installed on win/mac machine
       
* Using SSL/TLS is much more popular (Layer 5 (session) and up)
  * SSL: Secure Socket Layer
  * Deals with TLS (Transport Layer Security), Versions: TLS1, TLS1.1, TLS 1.2, DTLS (Datagram TLS (UDP packet within UDP packet): for real-time traffic like voice)
    
  * Client Solution: Full tunnel, looks like traditional IPsec
    * A client (host), that connects to VPN gateway (like ASA)
     
  * Clientless Solution: Using a web browser to establish a tunnel
    * You can then send traffic through the browser tunnel using an applet (like Port forwarding Java applet) but this is limited but great to for partner connectivity
      * establishes the connection to our ASA
      
  * HTTPS is not the only application supported
    * FTPS, POP3S, LDAPS, Wireless security (EAP-TLS), and others
  * Relies on certificates (installed on the server) to authenticate VPN peers

### Cisco AnyConnect Full SSL VPN Tunnel on Cisco ASA 
The Cisco AnyConnect VPN Client is now obsolete (past End-of-Life and End-of-Support status).

https://www.cisco.com/c/en/us/obsolete/security/cisco-anyconnect-vpn-client.html?dtid=osscdc000283

Hence, I won't show the configuration sample here.

## Clientless SSL VPN Wizard
There are different places where we might want to have clientless available for us which we don't have to give full access to:
* Business Partners
* Internet Kiosk
* Unmanaged Devices
Smart Tunnels is the way we do this:
* Supports for native application clients so you could open up an RDP connection
* Recommended for all applications without plug-ins