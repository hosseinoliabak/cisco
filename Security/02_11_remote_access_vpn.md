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

## Basic Cisco AnyConnect Full SSL VPN Tunnel on Cisco ASA Configuration
Uses:
* Self-signed or CA-signed identity certificate on Cisco ASA
  * If it is self-signed certificate:
    * For the first time you have to trust it and add it to your trusted certificate database OR
    * You can install the certificate ahead of time on the client
* Local user database on Cisco ASA
* Local address pool on Cisco ASA so that we can allocate the remote clients inside IP addresses
* Split tunneling and hairpin options on Cisco ASA to allow Internet access and use of their local network for clients
* How to authenticate the ASA itself (We can enrol with a CA and use Cisco ASA or Cisco ISE as a CA server to authenticate the client):
  * Temporary self-signed certificate generated by default
  * Persistent self-signed certificate
  * PKI-provisioned certificate recommended


