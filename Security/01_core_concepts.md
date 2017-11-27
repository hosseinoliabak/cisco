# Cryptography Basics
* Cryptology = Cryptography + Cryptanalysis
  * **Cryptography:**
    * Secure communication and data
    * Confidentiality
    * Data Integrity
    * Origin Authentication: Where the data is coming from
    * Non-repudiation
  * **Cryptanalysis:** the science of breaking (cracking) secret codes
    * Finding and exploiting weaknesses in cryptography algorithms

* **Encryption (Provides Confidentiality):** Hiding data from 3rd party. I have a plaintext message and by a "cipher algorithm" I hide it from everybody else. At the end of it we will get a Ciphertext.
We also want to do the process backward which is called decryption by that "cipher algorithm". This is called Confidentiality: I Encrypt a message and send it to you, then you decrypt it.
* **Hashing (Provides Integrity):** We also need the integrity. which is a one-way algorithm. We convert a plaintext to a Message Digest by a "hashing algorithm"

## Cryptanalysis
* **Brute-force attack:** The algorithm is quite easy and limited to the trial and error of as many character combinations as possible
* **Dictionary attack:** starts by a text file full of dictionary words. Dictionary is a table of commonly used passwords
* **Rainbow Table Attack:** pregenerated a bunch of hashes and the passwords from which they were calculated; it’s kind of index hash table; rainbow tables are really massive, could be terabytes
* **Known Plaintext attack:**
* **Chosen Plaintext attack:**
* **Cipthertext attack:** Usually against weakest encryption algorithms

## Hashing
Hash provides Integrity when it comes to the CIA of security
* Arbitrary-length data - Fixed-length hash
* One way
* Deterministic
* Hash Types
  * MD5: Invented in 1992; 128-bit hash. It is commonly used but not in governmental applications
  * SHA: Developed by NIS
    * SHA-1: 160-bit hash
    * SHA-2: Adopted by US Government. SHA-224, SHA-256, SHA-384, SHA-512
    * SHA-3: Standard released by NIST in 2015
  * RIPEMD: not very common, open standard; 128-, 160-, 256-, 320-bit digests
  * Collision: MD5 and SHA-1 prone to collision - when 2 different types of data generates the same hash
  * Hashes are for:
    * Password storage
    * Along with encryption

### Salting
The primary function of salts is to defend against dictionary* attacks or against its hashed equivalent, a pre-computed rainbow table attack by add something random to the password before being hashed\

| Password | Salt Value | String to be Hashed | Hashed Value = MD5 |
| --- | --- | --- | --- |
| pass123 | E1F53135E | pass123E1F53135E | ccc58be403dba6e6c39df39b07911cdc |

### Hash-based Message Authentication Code (HMAC) - Crypto Authentication with Hashes
HMAC uses existing hash functions and add a secret key as input to the hash function to provide authentication
along with integrity assurance. Then only the other party who also knows the secret key and can calculate the
resulting hash can correctly verify the hash. HMAC defeats man-in-the-middle attacks.

* Similar hashing for authentication of a peer, data integrity, and data authentication is used in multiple Cisco products
  * Routing update authentication: We want to make sure the routing update comes from a neighbor are not modified in transit. We don't want anybody injecting routes in our routing updates
  * IPsec gateways (ASA, Cisco IOS)
  * TACACS+ sessions for AAA

## Encryption
* Encryption algorithms are not secret!
* Keys used with the encryption algorithm create a unique cipher
* Symmetric encryption: Uses one key to encrypt and decrypt. The key need to be changed often
* Asymmetric encryption: Uses key pair (two keys) to encrypt and decrypt. Example: Public/Private keys

## Classical Encryption Algorithms

### Transposition based
Rearranging the letters in a string of text: i.e. HELLO -> OLLEH

### Substitution based

#### Monoalphabetic
Substitutes one character for another (i.e. A=C, B=D, C=E,…).
##### Caesar Cypher (shift cipher):
For example, here's the Caesar Cipher encryption of a message, using a right shift of 3 (ROT3):
* Plaintext:  THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG
* Ciphertext: QEB NRFZH YOLTK CLU GRJMP LSBO QEB IXWV ALD

To change the key we agree on a schedule, for example:
- On Monday: ROT4
- On Tuesday: ROT8
- On Wednesday: ROT13
- ... and so on

#### Polyalphabetic
Using multiple substitution alphabets:

##### Vigenère Cipher:
Like Caesar Cypher but added some complexity into it; Vigenère cipher is the sequence of Caesar ciphers with different transformations.
* Phrase: We Attack at Dawn; Key: Badge (XE DZXBCN GX EAZT)
* During the 16th century
* Vigenère Cipher supposedly the first cipher which used and encryption key. This ia where the idea of encryption keys comes from

##### Edward Hebern:
* 19th century
* An electro-mechanical contraption
* Used a single rotor, in which the secret key is embedded in a rotating disk. The key encoded a substitution table and each key press from the keyboard resulted in the output of cipher.

##### Enigma Machine
* Invented by German Engineer Arthur Scherbius
* Used 3 or 4 rotors
* Rotated at different rates
* You typed and the ciphertext was the result

#### Binary Encryption:
  * Phrase: MIKE (01001101 01001001 01001011 01000101); Algorithm (Exclusive OR): 11010. Then, we have:

| Phrase | 01001101 01001001 0100101101000101 |
| --- | --- |
| Key | 11010110 10110101 10101101 01101011 |
|__XOR Result__|__10011011 11111100 11100110 00101110__|

## Modern substitution based
The followings are modern Ciphers. I decided to keep the root titles style in heading 2.
## Symmetric-Key Algorithms
* Key lengths between 40 - 256 bits. Nowadays we don't use less than 80-bit keys
* Fast: Useful for VPN
* Key is a shared secret key that was negotiated ahead of time
* Key management is a challenge: must have common secret key prior to exchange
### Blocking ciphers
A block cipher is a deterministic algorithm operating on fixed-length groups of bits, called a block, with an unvarying transformation that is specified by a symmetric key.
#### Data Encryption Standard (DES):
* IBM formed a Crypto-Group, headed by Horst Fiestel
* Designed the Lucifer cipher
* 1973 NIST puts out an Request For Proposal
* Lucifer was eventually accepted and was called DES
First open standard. Does the following steps 16 times:
1.	Key: it drops the last 8 bits of the key with that we have 2x28-bit chunks (56 bits)
2.	We then grab the first 24 bits from each half, and put them together and we’ve created what’s known as a subkey, which is 48 bits
3.	first takes first 64-bit chunk of data from the data stream
4.	Initial Permutation: a very specific stirring of the data
5.	Fiestel function: split to 2x32 bits halves; then set one of the halves aside; we expand these 32 bits to a 48-bit chunk of data using an expansion function, then we apply XOR function using the subkey we have already generated. In DES, S boxes take in 64 bits, and output four bits. We apply the 8 different S boxes to the data creating a 32-bit output. We then put 2x32 bits together but backwards (Final permutation)
* Problems with DES:
  * Short key: 56 bits
  * In 1997 DES was broken by an exhastive search attack
#### Alternatives of DES
##### 3DES:
64-bit block size; 16 rounds; Key size: 3 times of DES process x56 bit= 168 bit

##### Blowfish:
64-bit block size; 16 rounds; Key size: variable 32 - 448 bits. Overshadowed by AES.

##### Advanced Encryption Standard (AES):
128-bit block size; 10, 12, or 14 rounds; Key size:128, 192, or 256 bits depending on rounds (WPA-2 Encryption)

#### Streaming ciphers
One bit at a time (pseudo-random)

##### Rivest's Cipher 4 (RC4):
1 bit at a time, 1 round, key size: 40-2048 bits (WPA Encryption, Useful for video and audio encryption because it encrypts bit by bit )

##### RC5:
These days is used for SSL and HTTPs

## Asymmetric-Key Algorithms (AKA Public key Cryptography)
Consist of key-pairs: We use 2 different keys that mathematically work together as a pair (Public Key and Private Key).
* **Private key** is never shared - must be kept secret
* **Public key** is freely shared
* Anything encrypted with the public key can be decrypted with the matching private key
* Anything encrypted with the private key can be decrypted with the public key (Digital Signature)
* Key length from 512 to 4096 bits

1. **Proof of Origin:** provides proof of original sender. This does not provide confidentiality as anyone could access the public key
2. **Add confidentiality to the Proof of Origin:** Senders borrow receiver's public key and encrypts the message then sends it to the receiver. The receiver party then can decrypt the message by his private key
3. Then, receiver borrows the senders public key to decrypt the original message which encrypted by sender's private key

Asymmetric encryption is slower than symmetric therefore it is not good for large amount of data. Is there a way to combine
Asymmetric and symmetric encryption together?

We can combine a asymmetric to initiate a secured and proof-of-origin and a confidential passage of message. Then we could
send a symmetric key between the two and switch to symmetric encryption (SSL and HTTPs)

### Key-based Asymmetric Encryption
#### Rivest, Shamir, and Adleman (RSA):
Published in 1977. Cornerstone is to take to large prime number and multiply them together to generate a big-semiprime number;
with that we have generated key pairs.

##### How an RSA key exchange takes place:
Alex and Bob want to communicate via RSA asymmetric encryption. First thing they’re going to generate each of their
own key pairs, and then they go about the process of exchanging their public keys. Now keep in mind if Eve grabs
one of these public keys, it doesn’t really make any difference because the only thing Eve could do would be encrypt
something. But there is a problem, and that is, what id Eve pretends to be Alice or Bob? Well the RSA guys thought
about that stuff ahead of time, and RSA includes all kind of protocols that do things that include what we call
authentication (we will discuss about authentication later but in fact it boils down to digital signatures and
certificates just to give you an idea).
* Problem with RSA is you need to use a large prime numbers - 1024-bit - or longer which become big keys

#### Elliptic Curve Cryptography (ECC):
* Goal: Reduce the overhead of calculation using large prime numbers
* Small key: a 3075-bit RSA key can now be replaced with roughly a 256-bit ECC key
* Elliptic curve formula: y2=x3+ax+b

### Non-key-based Asymmetric Encryption
Senders and receiver may have no access to a PKI solution
#### Diffie-Hellman (DH):
* Public key and private keys are not generated as pairs
* DH allows two devices that do not yet have a secure connection to establish shared secret keying material (keys that can be used with symmetrical
algorithms, such as AES)
* Complicated
* Sender and receiver negotiate a shared key. Each has public and private integers
* Public integers are exchanged and then calculated over several passes to derive shared-secret key
* With that both Bob and Alice can have the same symmetric key, without that key ever being moved across the wire
so that Eve could see it. Now to do this, we’re going to use DH. DH is a key exchange protocol
(AKA: key agreement protocol) and the whole goal of DH is to take advantage of what we call modulo arithmetic
  * 3? mod 17=2314692094782383457

##### How a DH works:
We want Alice and Bob to have the unique value; so, to do this we actually use asymmetric encryption here, and that first of all either Bob or Alice define a particular public key, this public key is a big long number. Both Alex and Bob generate a random private value then using this groovy mathematics, in essence they’re gonna mix these values together creating third value, then Alice and Bob exchange this mix, then Alice and Bob add their own private value to this mix and it creates the exact same value, and this is the number that we can go ahead and do symmetric encryption with, and Eve will never know what that number is.

#### ElGamal
* Based on DH
* Designed to create a complete PKI (cryptosystem)
* Available publically - not patented
* Disadvantage: Double the length of the message making large transfers difficult

### Digital Signatures
* Digital signatures represent a commitment to follow through, or prove you are who you say you are
* Data is large-expensive to encrypt with private key. Instead, digital signatures hash the data and encrypt the hash
* **Authentication:** User must authenticate to system to gain access to private key
* **Data integrity:** Message is hashed
* **No confidentiality by default:** Original message is not encrypted
* **Non-repudiation:** Typically, non-repudiation refers to the ability to ensure that a party to a contract or a
communication cannot deny the authenticity of their signature on a document or the sending of a message that
they originated. Achieved by:
  * Logging
  * Auditing

You never send public key by itself. To create a digital signature for a document,
you hash the document using your private key. Others can verify your digital signature with your public key.
Public Key + My Digital Signature + 3rd party who we trust’s Digital Signature are inside this digital certificate.

## Operate and Implement Cryptographic Systems
* How keys are managed?
* How are public keys distributed?
* How do you know you have the real public key?

* How keys are generated?
  * Modern systems create integers for both symmetric and asummetric keys. Keys can be automatically created by a
computer random number generator.. The computer random number generator is never really that random. So you can inject
or seed to increase the randomness.
  * Keys can be manually created, such as shared secret like the shared keys in RADUIS
* Key distribution:
  * Best to use modern Cryptosystems
  * Can be distributed in-band - over existing communication infrastructure
  * Can be Out-of-band - such as a handwritten note
  * Distributed keys need to be managed
* Key management:
  * Lifetime of a key
  * Creation, Revocation, Renewal, Deletion
* Key Storage and Recovery:
  * Can be stored in software or hadware solution
  * Key recovery performed by Key Recovery Agent (Escrow Agent)
There are 3 ways to do trust:
* Generate your own certificates (unsigned certificate) fantastic as long as both parties understand that there is no third-party vouching for you
* Web of trust: requires lots of maintenance (email certificates, hard drive encryption, …)
  * Web of trust is not centralized
  * You and your friend can come up with a trust using public/private key concept. They xan sign the certificate themselves as trusted
* Public Key Infrastructure (PKI) is a hierarchical method that starts off with root servers up at the top, intermediate servers (to take the load off of the CA), and goes down to users

### Public Key Infrastructure (PKI)
* PKI is centralized
* PKI is the most common cryptosystem to generate, distribute, and manage the keys
* PKI is based on X.509 (ITU-T) protocol that identifies the parts in PKI. X.509 isn’t even used anymore
* PKCS: invented by RSA corporation; de-facto standard for a lot of PKI systems
  * PKCS#1: RSA Cryptography Standard
  * PKCS#3: Diffie-Hellman key exchange
  * PKCS#10: This is a format of a certificate request sent to a CA that wants to receive its identity certificate. This type of request would include the public key for the entity desiring a certificate
  * PKCS#7: This is a format that can be used by a CA as a response to a PKCS#10 request. The response itself will very likely be the identity certificate (or certificates) that had been previously requested
  * PKCS#12: A format for storing both public and private keys using a symmetric passwordbased key to "unlock" the data whenever the key needs to be used or accessed

* Suppose I want to send a message to my friend Bob
1. I submit a Certificate Signing Request (CSR) to a CA server
* Certificate Authority (CA):
  * Comodo
  * Symantec (or VeriSign before it was purchased by Symantec)
  * GoDaddy
  * Microsoft Windows Certificate Services
  * Cisco IOS
  * Cisco ASA
  * Cisco ISE
2. CA sends back to me an Identity Certificate (the identity of the device such as its IP address, FQDN, and the public key of that device)
3. I also retrieve a root certificate (it is actually the CA's public key) from that CA server. The root certificate allows me to read the Identity certificate that the CA has signed
4. Bob is going to the same process
5. Now, we both have our own identification card which says this is who I am
6. Now I want to talk to Bob saying him hey Bob this is my identity. Bob looks at that identity which has the CA's digital signature. The only way Bob can read the digital signature is by using the root certificate (CA's public key)

## Internet Protocol Security (IPsec)
Everybody by the end of 1990s, was making their own security: SSL/TLS, SSH.
They asked what if instead of having all kind of these applications, we could come
up with some kind of end-to-end basis IP security. And this is called IPsec.
IPsec is not only one protocol, it is bunch of protocols.

IP security is an open standard; it is a collection of protocols and algorithms
used to protect IP packets at layer 3

IPsec has four fundamental goals:
* **Confidentiality:** provided through encryption changing clear text o cipher text
  * DES, 3DES, AES, RSA
* **Data Integrity:** provided through hashing or HMAC to verify data has not been manipulated during its transit
  * MD5, SHA-1, SHA-2
* **Peer authentication:** the sender and receiver will authenticate each other to make sure that we are really talking with the device we intend to
  * Pre-Shared Key (PSK): peers authenticate each other by comparing preconfigured secret key
  * RSA signature: peers authenticate each other using digital signature, the sending peer creates an encrypted hash using their private key, attaches it to a message and forwards it; the remote peer decrypts the hash using the public key and compares it with a recomputed hash
  * Elliptical Curve Digital Signature Algorithm (ECDSA): It is smaller than RSA signatures
* **Anti-replay:** verifies that each packet is unique and not duplicated. (By using sequence numbers, IPsec will not transmit any duplicate packets)

### IPsec protocol framework:
What gonna happens with data?
There are two primary methods for implementing IPsec to protect data in motion

#### Encapsulation Security Payload (ESP):
* Provides data authentication and integrity using HMAC MD5 or HMAC SHA-1
* Provides confidentiality using DES, 3DES, or AES
* Enforces anti-replay protection
* IP protocol 50. If NAT-T (UDP 4500) is in use, the ESP header is hidden behind a UDP header

#### Authentication Header (AH):
* Provides data authentication and integrity using HMAC MD5 or HMAC SHA-1
* Does NOT provide confidentiality (no encryption)
  * For that reason, we do not frequently see AH being used
* Problem with NAT
* IP protocol number 51

### ESP and AH runs at 2 very different modes:
**Transport mode** and **Tunnel mode**. The main difference between the two is that with transport mode we will use
the original IP header while in tunnel mode, we use a new IP header.

<img src="https://user-images.githubusercontent.com/31813625/33246997-c58763d2-d2e8-11e7-8b89-1912bb4fc2da.png" width="608" height="241" />

#### Transport Mode
* Keeping the original IP
* Protects the payload of the packet but leaves the original IP address in clear
* Works great if everybody had the same IP range and we got rid of NAT.
* Often between two hosts that want to protect some insecure traffic (example: telnet traffic)
* Security is only provided for transport layer and above
* Supports unicast traffic
* IPsec client remote-access

#### Tunnel Mode:
* Typically used for site-to-site VPNs where we need to encapsulate the original IP packet since these are mostly private IP addresses and can't be routed on the Internet
* Tunnel mode is most commonly used between gateways, or at a host to a gateway, the gateway acting as a proxy for the hosts behind it
* Provides security for the complete IP packet; the original packet is encrypted and then encapsulated in another IP packet
* Supports unicast traffic
* IPsec Site-to-Site VPN

![image](https://user-images.githubusercontent.com/31813625/32991394-bace8366-cd08-11e7-8e25-f0f040ecf8af.png)

## SSL and TLS Protocol Framework
* SSL as a protocol was originally developed by Netscape
* TLS and its predecessor SSL are cryptographic protocols that provide secure transactions on
the Internet for things such as e-mail, web browsing, instant messaging, and so on
* Most online transactions that are browser based are secured by SSL or TLS
* Both of these protocols provide confidentiality, integrity, and authentication services
* operate at the session layer and higher in the OSI logical model
* They both can use the public key infrastructure (PKI) and digital certificates for authentication of the VPN endpoints and for establishing
encryption keys that may be used
* Similar to IPsec, these protocols use symmetric algorithms for bulk encryption, and asymmetric algorithms are used for the authentication and
for the exchange of keys
* SSL 3.0 served as the basis of TLS 1.0
* Both begin with asymmetric encryption with certificate - then switch to symmetric
and shared secret key
* Type of VPNs:
  * SSL clientless remote access
  * SSL full-tunnel client remote-access

## Types of VPNs
* **IPsec:** Implements security of IP packets at Layer 3 of the OSI model, and can be used
for site-to-site VPNs and remote-access VPNs
* **SSL:** Secure Sockets Layer implements security of TCP sessions over encrypted SSL tunnels
of the OSI model, and can be used for remote-access VPNs (as well as being used to
securely visit a web server that supports it via HTTPS)
* **MPLS:** Multiprotocol Label Switching and MPLS Layer 3 VPNs are provided by a service
provider to allow a company with two or more sites to have logical connectivity
between the sites using the service provider network for transport. This is also a type of
VPN (called MPLS L3VPN), but there is no encryption by default. IPsec could be used
on top of the MPLS VPN to add confidentiality (through encryption) and the other benefits
of IPsec to protect the Layer 3 packets. The primary VPNs that provide
encryption, data integrity, authentication of who the peer is on the other end of the
VPN, and so on use IPsec or SSL.

## Secure/Multipurpose Internet Mail Extensions (S/MIME)
* Used in many mail application today
* Standard for public key encryption and signing of MIME data-email
* Provides authentication, non-repudiation, integrity, and message encryption