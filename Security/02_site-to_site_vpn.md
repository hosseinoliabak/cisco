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
