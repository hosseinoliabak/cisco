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
Smart Tunnels:
* Supports for native application clients so you could open up an RDP connection
* Recommended for all applications without plug-ins

### Configuration

#### Topology

![Topology](https://user-images.githubusercontent.com/31813625/33532835-15315460-d86b-11e7-99cb-469546662e59.png)

#### Wizard

![Wizard](https://user-images.githubusercontent.com/31813625/33532780-a611ff30-d86a-11e7-8649-6c975deff5b4.png)
![Connection](https://user-images.githubusercontent.com/31813625/33532781-a61e18f6-d86a-11e7-9b77-1627d6e1a448.png)
![Interface](https://user-images.githubusercontent.com/31813625/33533026-96844dc8-d86c-11e7-88b6-cd1c1b63118d.png)
![Manage Certificate](https://user-images.githubusercontent.com/31813625/33532783-a636d80a-d86a-11e7-9072-bafb8e8da5b9.png)
![Accessing Connection Profile](https://user-images.githubusercontent.com/31813625/33532784-a643225e-d86a-11e7-801e-c724da37515c.png)
![User Authentication](https://user-images.githubusercontent.com/31813625/33532785-a65035ca-d86a-11e7-9f95-e923bd373ed0.png)
![Group Policy](https://user-images.githubusercontent.com/31813625/33532786-a65bee88-d86a-11e7-822d-9c1e33df39a5.png)
![Bookmarks](https://user-images.githubusercontent.com/31813625/33532787-a6697d5a-d86a-11e7-801c-26b6f3f90767.png)
![Bookmark list](https://user-images.githubusercontent.com/31813625/33533027-969128d6-d86c-11e7-9bd9-7cefcad2f91f.png)
![Choose the bookmark](https://user-images.githubusercontent.com/31813625/33532789-a67fa9ea-d86a-11e7-941b-ea2a74002cb3.png)
![Confirm the bokmark](https://user-images.githubusercontent.com/31813625/33532790-a68d9a14-d86a-11e7-8be1-c712eea0d42b.png)
![Summary](https://user-images.githubusercontent.com/31813625/33532791-a69b46be-d86a-11e7-90f6-dca9638b3611.png)

#### Client Access

![Client Prompted](https://user-images.githubusercontent.com/31813625/33533235-bc05ba18-d86d-11e7-9a73-80d52bbe782b.png)
![Client Browses](https://user-images.githubusercontent.com/31813625/33533342-83238670-d86e-11e7-9445-c09cc8df7168.png)
![Client Accesses to Web Server 10.10.10.1](https://user-images.githubusercontent.com/31813625/33533343-8330204c-d86e-11e7-905f-e0fd8d96436b.png)

<pre>
!
interface GigabitEthernet0/0
 nameif outside
 security-level 0
 ip address 192.168.122.254 255.255.255.0
!
interface GigabitEthernet0/1
 nameif inside
 security-level 100
 ip address 10.10.10.254 255.255.255.0
!
!
ftp mode passive
dns domain-lookup outside
dns domain-lookup inside
pager lines 23
mtu outside 1500
mtu inside 1500

http server enable
http 0.0.0.0 0.0.0.0 inside

crypto ca trustpoint TrustPoint0
 enrollment self
 subject-name CN=ciscoasa
 ip-address 192.168.122.254
 crl configure
crypto ca trustpoint ASDM_TrustPoint0
 enrollment terminal
 subject-name CN=ciscoasa
 crl configure

crypto ca certificate chain _SmartCallHome_ServerCA
 certificate ca 6ecc7aa5a7032009b8cebcf4e952d491
    308205ec 308204d4 a0030201 0202106e cc7aa5a7 032009b8 cebcf4e9 52d49130
    0d06092a 864886f7 0d010105 05003081 ca310b30 09060355 04061302 55533117
    30150603 55040a13 0e566572 69536967 6e2c2049 6e632e31 1f301d06 0355040b
  quit

ssl trust-point TrustPoint0 outside
webvpn
 enable outside
 tunnel-group-list enable
 cache
  disable
 error-recovery disable
group-policy PARTNER_GROUP internal
group-policy PARTNER_GROUP attributes
 vpn-tunnel-protocol ssl-clientless
 webvpn
  url-list value PARTNER_BOOKMARKS
username test password P4ttSyrm33SV8TYp encrypted privilege 15
tunnel-group PARTNER type remote-access
tunnel-group PARTNER general-attributes
 default-group-policy PARTNER_GROUP
tunnel-group PARTNER webvpn-attributes
 group-alias Partner enable
 group-url https://192.168.122.254/Partner enable
!
policy-map global_policy
 class inspection_default
  inspect dns migrated_dns_map_1
  inspect ftp
  inspect h323 h225
  inspect h323 ras
  inspect ip-options
  inspect netbios
  inspect rsh
  inspect rtsp
  inspect skinny
  inspect esmtp
  inspect sqlnet
  inspect sunrpc
  inspect tftp
  inspect sip
  inspect xdmcp
!
prompt hostname context
no call-home reporting anonymous
</pre>