# DMVPN Over IPsec
To configure IPsec in this article, you should already configured DMVPN Phase 3. I used
iBGP for routing. (I preserved the last configuration from 08_14_dmvpn_ph3)
#### IPsec phase 1 ISAKMP tunnel
<pre>
HQ & Branches(config)#<b>crypto isakmp policy 10</b>
HQ & Branches(config-isakmp)#<b>authentication pre-share</b> 
HQ & Branches(config-isakmp)#<b>encryption aes 128</b>
HQ & Branches(config-isakmp)#<b>group 5</b>
HQ & Branches(config-isakmp)#<b>hash sha256</b>
HQ & Branches(config)#<b>crypto isakmp key DMVPN_KEY address 0.0.0.0</b></pre>

#### IPsec phase 2 (IPsec tunnel)
<pre>
HQ & Branches(config)#<b>crypto ipsec transform-set DMVPN_TRANSFORM esp-aes esp-sha-hmac</b>
HQ & Branches(cfg-crypto-trans)#<b>mode transport</b>
HQ & Branches(config)#<b>crypto ipsec profile DMVPN_PROFILE</b>
HQ & Branches(ipsec-profile)#<b>set transform-set DMVPN_TRANSFORM</b>
HQ & Branches(config)#<b>interface tunnel 0</b>
HQ & Branches(config-if)#<b>tunnel protection ipsec profile DMVPN_PROFILE</b></pre>

#### Verification
As IPsec occurs before DMVPN we should have a fresh start
<pre>
HQ & Branches(config)#<b>interface tunnel 0</b></pre>
<pre>
HQ(config-if)#<b>no shutdown</b></pre>
<pre>
Branches(config-if)#<b>no shutdown</b></pre>

<pre>
HQ#<b>show dmvpn | begin Hub</b>
Type:Hub, NHRP Peers:2, 

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 201.1.1.1              10.0.1.1    UP 00:01:43     D
     1 202.2.2.2              10.0.1.2    UP 00:01:42     D

HQ#<b>show crypto isakmp sa</b>
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
200.0.0.12      202.2.2.2       QM_IDLE           1008 ACTIVE
200.0.0.12      201.1.1.1       QM_IDLE           1007 ACTIVE
</pre>

<pre>
Branch1#<b>show crypto isakmp sa</b>
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
201.1.1.1       202.2.2.2       QM_IDLE           1002 ACTIVE
200.0.0.12      201.1.1.1       QM_IDLE           1006 ACTIVE
202.2.2.2       201.1.1.1       QM_IDLE           1004 ACTIVE

Branch1#<b>traceroute 192.168.2.2 source 192.168.1.1</b>
Type escape sequence to abort.
Tracing the route to 192.168.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.2 15 msec 15 msec 16 msec

Branch1#<b>show ip route</b> 
Gateway of last resort is 10.0.1.12 to network 0.0.0.0

B*    0.0.0.0/0 [200/0] via 10.0.1.12, 00:46:53
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.1.0/24 is directly connected, Tunnel0
L        10.0.1.1/32 is directly connected, Tunnel0
H        10.0.1.2/32 is directly connected, 00:44:35, Tunnel0
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, GigabitEthernet0/3
L        192.168.1.1/32 is directly connected, GigabitEthernet0/3
H     192.168.2.0/24 [250/255] via 10.0.1.2, 00:44:35, Tunnel0
S     200.0.0.0/24 is directly connected, GigabitEthernet0/1
      201.1.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        201.1.1.0/24 is directly connected, GigabitEthernet0/1
L        201.1.1.1/32 is directly connected, GigabitEthernet0/1
S     202.2.2.0/24 is directly connected, GigabitEthernet0/1

Branch1#<b>show crypto ipsec sa peer 202.2.2.2</b>

interface: Tunnel0
    Crypto map tag: Tunnel0-head-0, local addr 201.1.1.1

   protected vrf: (none)
   <b>local  ident (addr/mask/prot/port): (201.1.1.1/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (202.2.2.2/255.255.255.255/47/0)
   current_peer 202.2.2.2 </b>port 500
     PERMIT, flags={origin_is_acl,}
    #<b>pkts encaps: 3, #pkts encrypt: 3</b>, #pkts digest: 3
    #<b>pkts decaps: 3, #pkts decrypt: 3,</b> #pkts verify: 3
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 201.1.1.1, remote crypto endpt.: 202.2.2.2
     plaintext mtu 1458, path mtu 1500, ip mtu 1500, ip mtu idb GigabitEthernet0/1
     current outbound spi: 0xB067DC4D(2959596621)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x8A86633F(2324063039)
        transform: esp-aes esp-sha-hmac ,
        <b>in use settings ={Transport, }</b>
        conn id: 3, flow_id: SW:3, sibling_flags 80000000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4608000/2948)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)
      spi: 0xDF9429D6(3751029206)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Transport, }
        conn id: 7, flow_id: SW:7, sibling_flags 80004000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4213278/2952)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x805A639F(2153407391)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Transport, }
        conn id: 4, flow_id: SW:4, sibling_flags 80000000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4608000/2948)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)
      spi: 0xB067DC4D(2959596621)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Transport, }
        conn id: 8, flow_id: SW:8, sibling_flags 80004000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4213278/2952)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:
</pre> 
* GRE uses IP protocol 47
* With IPsec, HQ will have an SA with each branches.
* With large network that might be burden on the hub router
* One solution is to implement DMVPN over Group Encrypted Transport (GETVPN)
  * GETVPN is any-to-any VPN