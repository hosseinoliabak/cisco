# IPsec Site-to-Site VPN
* Connects entire networks to each other
* VPN hosts don't require VPN client software
* VPN gateway (ISR, ASA) is responsible for encapsulating and encrypting outbound traffic over the Internet to a peer VPN gateway
* Upon receipt, the peer VPN gateway decrypts the content and relays the packet toward the target host inside it’s private network

**VPN Building Blocks:**

![image](https://user-images.githubusercontent.com/31813625/32991401-d53d8d0a-cd08-11e7-84f1-cdf97a3040f0.png)

To establish an IPsec tunnel, we use a protocol called **IKE (Internet Key Exchange)**.
There are two phases to build an IPsec tunnel:
### IKE phase 1 (ISAKMP tunnel):
The IKE phase 1 tunnel is only used for **management traffic**.
Who Begins the Negotiation? The initiator sends over all of its IKE Phase 1 policies, and the other VPN peer looks at all of
those policies to see whether any of its own policies match the ones it just received. If there
is a matching policy, the recipient of the negotiations sends back information about which
received policy matches, and they use that matching policy for the IKE Phase 1 tunnel
* **Negotiation:** The two peers will negotiate about
hashing (MD5 or SHA), Authentication (PSK or Digital Certificates),
DH group (the strength of the key that is used in the key exchange process),
encryption (DES, 3DES, or AES), and lifetime. In this phase, an **ISAKMP (Internet Security Association and Key Management Protocol)** session is
established. This is also called the ISAKMP tunnel or IKE phase 1 tunnel. The collection of parameters that the two devices will use is called a **SA (Security Association)**.
* **DH Key Exchange:** They use the DH group (DH key size for the exchange) they agreed to during the
negotiations, and at the end of this key exchange, they both have symmetrical keying material
* **Authentication:** When the authentication is successful, we have
completed IKE phase 1. The end result is an ISAKMP tunnel which is bidirectional.
The authentication could be done either using a PSK or using RSA digital signatures
(depending on what they agreed to use in Negotiation step)

This IKE Phase 1 tunnel is not used to encrypt or protect
the end user's packets. To protect the end user's packets, (which is the entire goal for IPsec),
the two VPN devices build a second tunnel for the sole purpose of encrypting the end-user
packets.

In my years of working with students, this is where the confusion sometimes creeps in, because during the configuration
the students say to themselves, "Didn't I already specify the details for encryption
and hashing? Why is it asking for them again in the configuration?" The answer is that we
have to set up specific commands to specify the IKE Phase 1 policies, and we set up a different
set of similar commands for the IKE Phase 2 policy (including the component called a
transform set).

IKE phase 1 has 2 modes: **main mode**, which takes more packets, or **aggressive mode**, which
takes fewer packets and is considered less secure.

### IKE phase 2 (IPsec tunnel):
IKE phase 2 has only one mode which is called **quick mode**.
IKE phase 2 builds the actual IPsec tunnel.

R1 (the sender peer) uses the IKE Phase 2 tunnel and encrypts the packet and encapsulates the encrypted packet with a new IP header
that shows the source IP address as R1 and the destination address as R2. The Layer 4 protocol
would show as being Encapsulating Security Payload (ESP), which is reflected in the
IP header as protocol #50. When R2 receives this, R2 de-encapsulates the packet, sees that
it is ESP, and then proceeds to decrypt the original packet. Once decrypted, R2 forwards
the plaintext packet to the server.

Behind the scenes, the IKE Phase
2 tunnel really creates two one-way tunnels: one from R1 to R2 and one from R2 to R1.

Once IKE phase 2 is completed, we have an IPsec tunnel that we can use to
protect our user data. This user data will be sent through the IKE phase 2 tunnel

These tunnels are often referred to as the security agreements between the two VPN peers. Many times, these
agreements are called security associations (SA). Each SA is assigned a unique number for
tracking

### IPsec VPN negotiation steps:
1.	**Initiation:** An IPsec tunnel is initiated when a host sends "interesting" traffic to a destination host. Interesting traffic is distinguished by a crypto ACL
2.	**ISAKMP Tunnel:** The interesting traffic initiates IKE phase 1 between the peer routers. The IPsec peers negotiate the established IKE security association (SA) policy. Once the peers are authenticated, a secure tunnel is created by using ISAKMP
3.	**IPsec Tunnel:** The peers then proceed to IKE phase 2
4.	**Data Transfer:** The IPsec tunnel is created, the data is securely transferred
5.	**Termination:** The IPsec tunnel terminates when IPsec SAs are deleted or when their lifetime expires

### Configuration with Cisco IOS routers
Original IP packet will be encapsulated in a new IP packet and encrypted
before it is sent out of the network.
0. Before all, we have to have end-to-end connectivity
1. Make sure that out ACLs are compatible with our IPsec. For example:
  * `access-list 102 permit ahp host x host y`
  * `access-list 102 permit esp host x host y`
  * `access-list 102 permit udp host x host y eq isakmp`
  * `access-list 102 permit udp host x host y eq non500-isakmp`

1. Define the interesting traffic: the crypto ACL is not applied to an interface, it's gonna be applied to a crypto map (step 4), then the crypto map
will be applied to the outgoing interface. If it is interesting, then it will be encrypted. If not interested, it will bypass the encryption.
    <pre>
    R1(config)#<b>ip access-list extended VPN-TRAFFIC</b>
    R1(config-ext-nacl)#<b>permit ip 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255</pre></b>

2. ISAKMP Policy Parameters:

    | Parameter | Keyword | Accepted Values | Default Value |
    | --- | --- | --- | --- |
    |Encryption | des<br /> aes<br /> aes 192<br /> aes 256<br />| 56-bit DES-CBC<br /> 128-bit AES<br /> 192-bit AES<br /> 256-bit AES | des |
    | Hash | sha<br /> md5 | SHA-1 (HMAC variant)<br /> MD5 (HMAC variant)<br /> | sha |
    | Authentication | rsa-sig<br /> rsa-encr<br /> pre-share | RSA signatures<br /> RDA encrypted nonces<br /> Preshared keys | rsa-sig |
    | Group | 1<br /> 2<br /> 5 | 768-bit DH<br />  1024-bit DH<br /> 1536-bit DH | 1 |
    | Lifetime | seconds | Any number of seconds | 86,400 sec |

  * We can see the default policies by `show crypto isakmp policy`
  * If we want to change the default values, we can use global `crypto isakmp policy <priority#>` command. The lower priority wins. If the first policy matches on both sides, the IOS don't look at the rest. So we put the strongest the lowest.
  * Don't forget the pre-shared key:
    * `crypto isakmp key cisco123 address <the peer address>`

3. Configure IPsec Transform Set (Quick Mode): combination of hash and encryption algorithm for users traffic
  * Transform sets: Mechanism for payload authentication + mechanism for payload encryption + IPsec mode (tunnel mode is default)
  * Create a transformset stronger that ISAKMP. For example if you chose AES 192 in Phase1, here choose AES 192 or higher, not DES
  * We can have more than 1 transform set. For example, below, transform-set B on R1 matches with transform-set C on R3
     * (R1):
        * transform-set A: esp-aes 192, esp-sha-hmac, tunnel
        * transform-set B: esp-aes, esp-sha-hmac, tunnel
        * transform-set C: esp-3des 192, esp-md5-hmac, tunnel
     * (R3):
        * transform-set A: esp-aes 256, esp-sha-hmac, tunnel
        * transform-set B: esp-aes, esp-md5-hmac, tunnel
        * transform-set C: esp-aes, esp-sha-hmac, tunnel
  * For example both sides should have:
    <pre>
    R1(config)#<b>crypto ipsec transform-set A esp-aes 192 esp-sha-hmac</b>
    R1(cfg-crypto-trans)#<b>mode tunnel</b>
    R1(cfg-crypto-trans)#<b>exit</b></pre>

4. Crypto map: What to protect (match address, crypto ACL), How to protect (set transform-set), Where to send (set peer)
    <pre>
    R1(config)#<b>crypto map VPN 10 ipsec-isakmp</b>
    % NOTE: This new crypto map will remain disabled until a peer
      and a valid access list have been configured.
    R1(config-crypto-map)#<b>set peer 209.165.202.129</b>
    R1(config-crypto-map)#<b>set transform-set A</b>
    R1(config-crypto-map)#<b>match address VPN-TRAFFIC</b></pre>

* Now we activate the crypto map on the egress interface:
    <pre>
    R1(config)#<b>interface GigabitEthernet0/2</b>
    R1(config-if)#<b>crypto map VPN</b>
    *Nov 27 23:53:03.070: %CRYPTO-6-ISAKMP_ON_OFF: ISAKMP is ON
    </pre>





