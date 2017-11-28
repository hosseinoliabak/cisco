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

<img src="https://user-images.githubusercontent.com/31813625/33339989-c85ce508-d448-11e7-953a-e90ea5faea3c.png" width="658" height="338" />

0.
  * Before all, we have to have end-to-end connectivity
  * Make sure that if you have ACLs in your routers, ACLs are compatible with your IPsec. For example:
    * `access-list 102 permit ahp host x host y`
    * `access-list 102 permit esp host x host y`
    * `access-list 102 permit udp host x host y eq isakmp`
    * `access-list 102 permit udp host x host y eq non500-isakmp`
    * In the following example, we don't have any previous ACLs so we skip this step

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
    *Nov 27 23:53:03.070: %CRYPTO-6-ISAKMP_ON_OFF: ISAKMP is ON    </pre>

Try it yourself: Note that unnecessary output omitted from show running-config
#### R1:
<pre>
!
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 2
crypto isakmp key cisco address 209.165.202.129
!
crypto ipsec transform-set A esp-des esp-md5-hmac
 mode tunnel
!
crypto map VPN 10 ipsec-isakmp
 set peer 209.165.202.129
 set transform-set A
 match address VPN-TRAFFIC
!

interface GigabitEthernet0/2
 ip address 209.165.200.225 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
 crypto map VPN
!
interface GigabitEthernet0/3
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/2
ip route 209.165.202.128 255.255.255.252 GigabitEthernet0/2
!
ip access-list extended VPN-TRAFFIC
 permit ip 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255
!
</pre>

#### R2:
<pre>
!
interface GigabitEthernet0/1
 ip address 209.165.200.226 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 ip address 209.165.202.130 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
ip route 192.168.1.0 255.255.255.0 GigabitEthernet0/1
ip route 192.168.3.0 255.255.255.0 GigabitEthernet0/3
!
</pre>

#### R3:
<pre>
!
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 2
crypto isakmp key cisco address 209.165.200.225
!
crypto ipsec transform-set A esp-des esp-md5-hmac
 mode tunnel
!
crypto map VPN 10 ipsec-isakmp
 set peer 209.165.200.225
 set transform-set A
 match address VPN-TRAFFIC
!
interface GigabitEthernet0/2
 ip address 209.165.202.129 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
 crypto map VPN
!
interface GigabitEthernet0/3
 ip address 192.168.3.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
ip route 192.168.1.0 255.255.255.0 GigabitEthernet0/2
ip route 209.165.200.224 255.255.255.252 GigabitEthernet0/2
!
ip access-list extended VPN-TRAFFIC
 permit ip 192.168.3.0 0.0.0.255 192.168.1.0 0.0.0.255
!
</pre>

### Verification
<pre>
R1#<b>clear crypto sa</b>
R1#<b>debug crypto isakmp</b>
R1#<b>debug crypto ipsec</b></pre>
IKE Phase 1 Configuration:
<pre>
R1#<b>show crypto isakmp policy</b>

Global IKE policy
Protection suite of priority 10
  encryption algorithm:  AES - Advanced Encryption Standard (128 bit keys).
  hash algorithm:    Secure Hash Standard
  authentication method:  Pre-Shared Key
  Diffie-Hellman group:  #2 (1024 bit)
  lifetime:    86400 seconds, no volume limit
</pre>
IKE Phase 1 Verification
<pre>
R1#<b>show crypto isakmp sa</b>
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
209.165.202.129 209.165.200.225 QM_IDLE           1002 ACTIVE

IPv6 Crypto ISAKMP SA</pre>
Transform-set Configuration
<pre>
R1#<b>show crypto isakmp sa</b>
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
209.165.202.129 209.165.200.225 QM_IDLE           1002 ACTIVE

IPv6 Crypto ISAKMP SA
</pre>
Crypto map Configuration
<pre>
R1#<b>show crypto map</b>
  Interfaces using crypto map NiStTeSt1:

Crypto Map IPv4 "VPN" 10 ipsec-isakmp
  Peer = 209.165.202.129
  Extended IP access list VPN-TRAFFIC
      access-list VPN-TRAFFIC permit ip 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255
      access-list VPN-TRAFFIC permit icmp 192.168.1.0 0.0.0.255 192.168.3.0 0.0.0.255
  Current peer: 209.165.202.129
  Security association lifetime: 4608000 kilobytes/3600 seconds
  Responder-Only (Y/N): N
  PFS (Y/N): N
  Mixed-mode : Disabled
  Transform sets={
    A:  { esp-des esp-md5-hmac  } ,
  }
  Interfaces using crypto map VPN:
    GigabitEthernet0/2</pre>
IKE Phase 2 Verification
<pre>
R1#<b>show crypto ipsec sa</b>

interface: GigabitEthernet0/2
    Crypto map tag: VPN, local addr 209.165.200.225

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.1.0/255.255.255.0/1/0)
   remote ident (addr/mask/prot/port): (192.168.3.0/255.255.255.0/1/0)
   current_peer 209.165.202.129 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 0, #pkts encrypt: 0, #pkts digest: 0
    #pkts decaps: 0, #pkts decrypt: 0, #pkts verify: 0
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 209.165.200.225, remote crypto endpt.: 209.165.202.129
     plaintext mtu 1500, path mtu 1500, ip mtu 1500, ip mtu idb GigabitEthernet0/2
     current outbound spi: 0x0(0)
     PFS (Y/N): N, DH group: none

     inbound esp sas:

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:

     outbound ah sas:

     outbound pcp sas:

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.1.0/255.255.255.0/0/0)
   remote ident (addr/mask/prot/port): (192.168.3.0/255.255.255.0/0/0)
   current_peer 209.165.202.129 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 0, #pkts encrypt: 0, #pkts digest: 0
    #pkts decaps: 0, #pkts decrypt: 0, #pkts verify: 0
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 209.165.200.225, remote crypto endpt.: 209.165.202.129
     plaintext mtu 1446, path mtu 1500, ip mtu 1500, ip mtu idb GigabitEthernet0/2
     current outbound spi: 0xE7CB46BA(3888858810)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x59DC848B(1507624075)
        transform: esp-des esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 5, flow_id: SW:5, sibling_flags 80004040, crypto map: VPN
        sa timing: remaining key lifetime (k/sec): (4302002/3214)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0xE7CB46BA(3888858810)
        transform: esp-des esp-md5-hmac ,
        in use settings ={Tunnel, }
        conn id: 6, flow_id: SW:6, sibling_flags 80004040, crypto map: VPN
        sa timing: remaining key lifetime (k/sec): (4302002/3214)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:
R1#
*Nov 28 19:23:12.853: IPSEC(sibling_update_flow_stats): IPSEC: MIB Stats Ptr 0x1020CF68

*Nov 28 19:23:12.854: IPSEC(sibling_update_flow_stats): IPSEC: MIB Stats Ptr 0x1020CF68

*Nov 28 19:23:12.854: IPSEC(sibling_update_flow_stats): IPSEC: Flow ID : 0x14000005, Flow Stats Ptr 0xF06D580

*Nov 28 19:23:12.855: IPSEC(sibling_update_flow_stats): IPSEC: MIB Stats Ptr 0x1020CF68
</pre>
To see the current sessions:
<pre>
R1#<b>show crypto session</b>
Crypto session current status

Interface: GigabitEthernet0/2
Session status: UP-ACTIVE
Peer: 209.165.202.129 port 500
  Session ID: 0
  IKEv1 SA: local 209.165.200.225/500 remote 209.165.202.129/500 Active
  IPSEC FLOW: permit 1 192.168.1.0/255.255.255.0 192.168.3.0/255.255.255.0
        Active SAs: 0, origin: crypto map
  IPSEC FLOW: permit ip 192.168.1.0/255.255.255.0 192.168.3.0/255.255.255.0
        Active SAs: 2, origin: crypto map
</pre>