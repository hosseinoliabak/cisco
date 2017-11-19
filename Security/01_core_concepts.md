# Core Security Concepts
## Cryptography Basics
Hiding data from 3rd party.
### Caesar Cypher (shift cipher): For example, here's the Caesar Cipher encryption of a message, using a right shift of 3:
* Plaintext:  THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG
* Ciphertext: QEB NRFZH YOLTK CLU GRJMP LSBO QEB IXWV ALD

### Vigenère Cipher:
Like Caesar Cypher but added some complexity into it; Vigenère cipher is the sequence of Caesar ciphers with different transformations.
* Phrase: We Attack at Dawn; Key: Badge (XE DZXBCN GX EAZT)

### Binary Encryption:
  * Phrase: MIKE (01001101 01001001 01001011 01000101); Algorithm (Exclusive OR): 11010. Then, we have:

| Phrase | 01001101 01001001 0100101101000101 |
| --- | --- |
| Key | 11010110 10110101 10101101 01101011 |
|__XOR Result__|__10011011 11111100 11100110 00101110__|

## Hashing
Hash provides Integrity when it comes to the CIA of security
* fixed value
* One way
* Deterministic
* Hash Types
  * MD5: Invented in 1992; 128-bit hash
  * SHA: Developed by NIS
    * SHA-1: 160-bit hash
    * SHA-2: SHA-256, SHA-512
  * RIPEMD: not very common, open standard; 128-, 160-, 256-, 320-bit digests
  * Collision: MD5 and SHA-1 prone to collision - when 2 different types of data generates the same hash
  * Hashes are for:
    * Password storage
    * Along with encryption

## Hash-based Message Authentication Code – HMAC
HMAC uses existing hash functions and add a secret key as input to the hash function to provide authentication along with integrity assurance. Then only the other party who also knows the secret key and can calculate the resulting hash can correctly verify the hash. HMAC defeats man-in-the-middle attacks.

## Symmetric-Key Algorithms
### Blocking ciphers:
A block cipher is a deterministic algorithm operating on fixed-length groups of bits, called a block, with an unvarying transformation that is specified by a symmetric key.
#### Data Encryption Standard (DES):
First open standard. Do the following steps 16 times:
1.	Key: it drops the last 8 bits of the key with that we have 2x28-bit chunks (56 bits)
2.	We then grab the first 24 bits from each half, and put them together and we’ve created what’s known as a subkey, which is 48 bits
3.	first takes first 64-bit chunk of data from the data stream
4.	Initial Permutation: a very specific stirring of the data
5.	Fiestel function: split to 2x32 bits halves; then set one of the halves aside; we expand these 32 bits to a 48-bit chunk of data using an expansion function, then we apply XOR function using the subkey we have already generated. In DES, S boxes take in 64 bits, and output four bits. We apply the 8 different S boxes to the data creating a 32-bit output. We then put 2x32 bits together but backwards (Final permutation)
* Problems with DES:
  * Short key: 56 bits
#### Alternatives of DES:

##### Blowfish:
64-bit block size; 16 rounds; Key size: variable 32 - 448 bits
##### 3DES:
64-bit block size; 16 rounds; Key size: 3 times of DES process x56 bit= 168 bit

##### Advanced Encryption Standard (AES):
  * 128-bit block size; 10, 12, or 14 rounds; Key size:128, 192, or 256 bits depending on rounds (WPA-2 Encryption)

### Streaming ciphers:
One bit at a time (pseudo-random)
* Rivest Cipher 4 (RC4): 1 bit at a time, 1 round, key size: 40-2048 bits (WPA Encryption)

## Asymmetric-Key Algorithms
Consist of key-pairs: We use 2 different keys that mathematically work together as a pair (Public Key and Private Key). Because of the computational complexity of asymmetric encryption, it is usually used only for small blocks of data, typically the transfer of a symmetric encryption key (e.g. a session key). This symmetric key is then used to encrypt the rest of the potentially long message sequence.

### Rivest, Shamir, and Adleman (RSA):
Cornerstone is to take to large prime number and multiply them together to generate a big-semiprime number; with that we have generated key pairs.
#### How an RSA key exchange takes place:
Alex and Bob want to communicate via RSA asymmetric encryption. First thing they’re going to generate each of their own key pairs, and then they go about the process of exchanging their public keys. Now keep in mind if Eve grabs one of these public keys, it doesn’t really make any difference because the only thing Eve could do would be encrypt something. But there is a problem, and that is, what id Eve pretends to be Alice or Bob? Well the RSA guys thought about that stuff ahead of time, and RSA includes all kind of protocols that do things that include what we call authentication (we will discuss about authentication later but in fact it boils down to digital signatures and certificates just to give you an idea).
* Problem with RSA is you need to use a 2048-bit key or longer which become big keys

### Elliptic Curve Cryptography (ECC):
* Small key: a 3075-bit RSA key can now be replaced with roughly a 256-bit ECC key
* elliptic curve formula: y2=x3+ax+b

### Diffie-Hellman (DH):
Roughly in the same time as RSA
* Public key and private keys are not generated as pairs; it’s the situation where you just connect to somebody every now and then and you don’t want all extra rigmarole
* Complicated
* With that both Bob and Alice can have the same symmetric key, without that key ever being moved across the wire so that Eve could see it. Now to do this, we’re going to use DH. DH is a key exchange protocol (AKA: key agreement protocol) and the whole goal of DH is to take advantage of what we call modulo arithmetic
  * 3? mod 17=2314692094782383457

#### How an DH works:
We want Alice and Bob to have the unique value; so, to do this we actually use asymmetric encryption here, and that first of all either Bob or Alice define a particular public key, this public key is a big long number. Both Alex and Bob generate a random private value then using this groovy mathematics, in essence they’re gonna mix these values together creating third value, then Alice and Bob exchange this mix, then Alice and Bob add their own private value to this mix and it creates the exact same value, and this is the number that we can go ahead and do symmetric encryption with, and Eve will never know what that number is.
