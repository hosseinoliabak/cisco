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

![0](https://user-images.githubusercontent.com/31813625/33532835-15315460-d86b-11e7-99cb-469546662e59.png)
![1](https://user-images.githubusercontent.com/31813625/33532780-a611ff30-d86a-11e7-8649-6c975deff5b4.png)
![2](https://user-images.githubusercontent.com/31813625/33532781-a61e18f6-d86a-11e7-9b77-1627d6e1a448.png)
![3](https://user-images.githubusercontent.com/31813625/33532782-a62a0440-d86a-11e7-961e-e411fda5100b.png)
![5](https://user-images.githubusercontent.com/31813625/33532783-a636d80a-d86a-11e7-9072-bafb8e8da5b9.png)
![6](https://user-images.githubusercontent.com/31813625/33532784-a643225e-d86a-11e7-801e-c724da37515c.png)
![7](https://user-images.githubusercontent.com/31813625/33532785-a65035ca-d86a-11e7-9f95-e923bd373ed0.png)
![8](https://user-images.githubusercontent.com/31813625/33532786-a65bee88-d86a-11e7-822d-9c1e33df39a5.png)
![9](https://user-images.githubusercontent.com/31813625/33532787-a6697d5a-d86a-11e7-801c-26b6f3f90767.png)
![10](https://user-images.githubusercontent.com/31813625/33532788-a674e3f2-d86a-11e7-86a8-8befaaa6a164.png)
![11](https://user-images.githubusercontent.com/31813625/33532789-a67fa9ea-d86a-11e7-941b-ea2a74002cb3.png)
![12](https://user-images.githubusercontent.com/31813625/33532790-a68d9a14-d86a-11e7-8be1-c712eea0d42b.png)
![13](https://user-images.githubusercontent.com/31813625/33532791-a69b46be-d86a-11e7-90f6-dca9638b3611.png)
![14](https://user-images.githubusercontent.com/31813625/33532792-a6a6400a-d86a-11e7-9e06-4fa7413666c4.png)
![15](https://user-images.githubusercontent.com/31813625/33532793-a6b246ca-d86a-11e7-8423-742c2a99ee2b.png)
