# Configuring WAN Technologies

<img src="https://user-images.githubusercontent.com/31813625/33683978-ab591fb4-da9a-11e7-8b8f-f0018e5921e2.png" width="630" height="423" />

## Point-to-Point Protocol (PPP)
* RFC 1661
* Ethernet vs PPP:
  * Ethernet provides multiple access while PPP is point-to-point
  * Ethernet: Broadcast - PPP: Party of two
* PPP operates on the data link layer (layer 2) but the data link layer has been split into two pieces:
  * **NCP: Network Control Protocol:**  NCP will make sure you can run different
  protocols over our PPP link like IP, IPv6 but also CDP (Cisco Discovery Protocol)
  and older protocols like IPX or AppleTalk.
  * **LCP: Link Control Protocol:** LCP takes care of setting up the link
  If you enable authentication for PPP it will take care of authentication.
  Once the link has been setup we use NCP.
* If the link is serial the default encapsulation on Cisco routers is HDLC
you have to change it by `Router(config-if)#encapsulation ppp` command.
### How PPP Operates
1. Link dead: PPP is waiting for PHY to become active
2. Link establishment (LCP): initiates connection between the 2 endpoints
3. Authentication (optional): One of the end points authenticate the other end
4. Network-layer protocol (NCP):  See above
5. Link termination: in case of an error like authentication problem

### PPP Authentication
* It is a one-way authentication
* If you want to use authentication for PPP you have two options:
* Password Authentication Protocol (PAP):
  * Password is sent in clear
* Challenge Handshake Authentication Protocol (CHAP):
  * Password never sent in clear

### Configuration
This is an example where R1 authenticates R2 using CAHP while R2 authenticates R1 using PAP
#### R1
<pre>
R1(config)#<b>username Router2 password class</b>
R1(config)#<b>interface serial 1/1</b>
R1(config-if)#<b>encapsulation ppp</b>
R1(config-if)#<b>ppp authentication chap</b>
R1(config-if)#<b>ppp pap sent-username Router1</b>
R1(config-if)#<b>ip address 192.168.9.1 255.255.255.0</b></pre>

#### R2
<pre>
R2(config)#<b>username Router1 password cisco</b>
R2(config)#<b>interface serial 1/1</b>
R2(config-if)#<b>encapsulation ppp</b>
R2(config-if)#<b>ppp authentication pap</b>
R2(config-if)#<b>ppp chap hostname Router2</b>
R2(config-if)#<b>ppp chap password class</b>
R2(config-if)#<b>ip address 192.168.9.2 255.255.255.0</b></pre>

### Verification
<pre>
R1#show ppp interface serial 1/1
PPP Serial Context Info
—---------------—
Interface        : Se1/1
PPP Serial Handle: 0x8000000A
PPP Handle       : 0x6400000A
SSS Handle       : 0xFD00000A
AAA ID           : 21
Access IE        : 0x8A00000A
SHDB Handle      : 0x0
State            : Up
Last State       : Binding
Last Event       : LocalTerm

PPP Session Info
—------------—
Interface        : Se1/1
PPP ID           : 0x6400000A
Phase            : UP
Stage            : Local Termination
Peer Name        : Router2
Peer Address     : 192.168.9.2
Control Protocols: LCP[Open] CHAP+ IPCP[Open] CDPCP[Open]
Session ID       : 10
AAA Unique ID    : 21
SSS Manager ID   : 0xFD00000A
SIP ID           : 0x8000000A
PPP_IN_USE       : 0x11

Se1/1 LCP: <b>[Open]</b>
<b>Our Negotiated Options</b>
Se1/1 LCP:    <b>AuthProto CHAP (0x0305C22305)</b>
Se1/1 LCP:    MagicNumber 0x01102191 (0x050601102191)
<b>Peer's Negotiated Options</b>
Se1/1 LCP:    <b>AuthProto PAP (0x0304C023)</b>
Se1/1 LCP:    MagicNumber 0x02103025 (0x050602103025)

Se1/1 IPCP: [Open]
Our Negotiated Options
Se1/1 IPCP:    Address 192.168.9.1 (0x0306C0A80901)
Peer's Negotiated Options
Se1/1 IPCP:    Address 192.168.9.2 (0x0306C0A80902)

Se1/1 CDPCP: [Open]
Our Negotiated Options
  NONE
Peer's Negotiated Options
  NONE
</pre>
<pre>
R2#show ppp interface serial 1/1
PPP Serial Context Info
—---------------—
Interface        : Se1/1
PPP Serial Handle: 0x2C000009
PPP Handle       : 0xE000009
SSS Handle       : 0x79000009
AAA ID           : 20
Access IE        : 0x11000009
SHDB Handle      : 0x0
State            : Up
Last State       : Binding
Last Event       : LocalTerm

PPP Session Info
—------------—
Interface        : Se1/1
PPP ID           : 0xE000009
Phase            : UP
Stage            : Local Termination
Peer Name        : Router1
Peer Address     : 192.168.9.1
Control Protocols: LCP[Open] PAP+ IPCP[Open] CDPCP[Open]
Session ID       : 9
AAA Unique ID    : 20
SSS Manager ID   : 0x79000009
SIP ID           : 0x2C000009
PPP_IN_USE       : 0x11

Se1/1 LCP: <b>[Open]</b>
<b>Our Negotiated Options</b>
Se1/1 LCP:    <b>AuthProto PAP (0x0304C023)</b>
Se1/1 LCP:    MagicNumber 0x02103025 (0x050602103025)
<b>Peer's Negotiated Options</b>
Se1/1 LCP:    <b>AuthProto CHAP (0x0305C22305)</b>
Se1/1 LCP:    MagicNumber 0x01102191 (0x050601102191)

Se1/1 IPCP: [Open]
Our Negotiated Options
Se1/1 IPCP:    Address 192.168.9.2 (0x0306C0A80902)
Peer's Negotiated Options
Se1/1 IPCP:    Address 192.168.9.1 (0x0306C0A80901)

Se1/1 CDPCP: [Open]
Our Negotiated Options
  NONE
Peer's Negotiated Options
  NONE
</pre>

## PPPoE
* RFC 2516
* PPP frames encapsulated inside Ethernet frames
* PPPoE Provides PPP features over Ethernet:
  * Authentication
  * Dynamic layer 3 addressing
* PPPoE uses a client-server relationship
* PPPoE Stages:
  * Discovery:
    1. PPPoE client sends an Ethernet broadcast called Active Discovery Initiation (PADI)
    2. PPPoE server responds with Active Discovery Offer (PADO)
    3. The client sends a unicast Active Discovery Request (PADR) asking the server let's start a PPPoE session
    4. Servers sends an Active Discovery Session Confirmation (PADS) and then the PPPoE session is established
  * PPP Session
    * PPP frames are encapsulated in unicast Ethernet frames
    * PPP authentication happens **after** the PPPoE discovery stage