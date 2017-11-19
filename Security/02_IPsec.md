# Internet Protocol Security (IPsec)
Everybody by the end of 1990s, everybody was making their own security: SSL/TLS, SSH. They asked what if instead of having all kind of these applications, we could come up with some kind of end-to-end basis IP security. And this is called IPsec. IPsec is not only one protocol, it is bunch of protocols.

IPsec runs at 2 very different modes: The main difference between the two is that with transport mode we will use the original IP header while in tunnel mode, we use a new IP header.
* **Transport Mode:** keeping the original IP; works great if everybody had the exact same IP range and we got rid of NAT etc. Transport mode is often between two devices that want to protect some insecure traffic (example: telnet traffic).
* **Tunnel Mode:** is typically used for site-to-site VPNs where we need to encapsulate the original IP packet since these are mostly private IP addresses and can’t be routed on the Internet.

**What gonna happens with data? (IPsec framework protocols)**
* **Authentication Headers (AH):** IPsec packets have the Layer 4 protocol of AH (IP Protocol #51) which can do many parts of the IPsec objectives (by inserting HMAC into the AH), except for the important one of encryption of the data. For that reason, we do not frequently see AH being used
* **Encapsulating Security Payloads (ESP):** IPsec packets have the Layer 4 protocol of ESP (IP Protocol #50) which is encapsulated by the sender and decapsulated by the receiver for each IPsec packet. ESP is normally used instead of Authentication Header (AH). The ESP header is hidden behind a UDP header if NAT-T is in use

![image](https://user-images.githubusercontent.com/31813625/32991394-bace8366-cd08-11e7-8e25-f0f040ecf8af.png)

![image](https://user-images.githubusercontent.com/31813625/32991401-d53d8d0a-cd08-11e7-84f1-cdf97a3040f0.png)

### Internet Security Association and Key Management Protocol ISAKMP:
Creates a Security Association (SA) between two hosts. So, if two hosts want to start talking IPsec, they begin negotiation using ISAKMP. ISAKMP handle all kinds of stuff:
* Initial Authentication: We can use certificates, pre-shared keys
* Key Exchanges
* …

### The tunneling aspect of IPsec makes it natural for VPNs
* Pure IPsec VPN
* IPsec with L2TP: tunnel within a tunnel
