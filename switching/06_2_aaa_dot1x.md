# Authentication, Authorization and Accounting (AAA)
* Authentication: Who are you?
* Authorization: What do you have access to?
* Accounting: To generate logs.

Two network protocols providing this functionality are particularly popular: the RADIUS protocol (or its newer Diameter counterpart), and TACACS+ (Cisco proprietary )
* RADIUS: (UDP ports: auth, aut: 1812, 1645; Acc: 1813, 1646)
  * Radius Server: provides authentication
  * Radius Client (the switch): is not the one who is being authenticated; its job is to handle authentication request from supplicant
  * Radius Supplicant (the user, a laptop, a PC): could be a laptop which will be authenticated
* TACACS+: Cisco proprietary (auth, aut, acc: TCP 49); good at managing a bunch of devices
  * TACACS+ Server: provides authentication
  * TACACS+ Client: to handle authentication request from user
  * TACACS+ User: a laptop or a PC

<img src="https://user-images.githubusercontent.com/31813625/32981632-13806ec0-cc46-11e7-9b6c-4b3f5ba64c22.png" width="504" height="179" />

There are many different RADIUS servers you can use, for example:
* Cisco ACS (Cisco’s RADIUS and TACACS+ server software)
* Microsoft IAS (you can install it on Windows server 2003 or 2008).
* Freeradius (very powerful and free)
* Integrated in network devices (Cisco’s Wireless LAN controller have RADIUS server software for example).

### Local authentication:
<pre>
SW1(config)#<b>aaa new-model</b>
SW1(config)#<b>username test privilege 15 password test</b>
SW1(config)#<b>aaa authentication login default local-case</b>
SW1(config)#<b>aaa authentication login CONSOLE local</b>
SW1(config)#<b>line console 0</b>
SW1(config-line)#<b>login authentication CONSOLE</b></pre>

### Server-based authentication:
<pre>
SW1(config)#<b>aaa new-model</b>
SW1(config)#<b>aaa authentication login default group radius local</b></pre>
Then for RADIUS:
<pre>
SW1(config)#<b>radius-server host 192.168.1.200 key test</b></pre>
For TACACS+:
<pre>
SW1(config)#<b>tacacs-server host 192.168.1.200 key test</b></pre>
<pre>
SW1(config)#<b>line console 0</b>
SW1(config-line)#<b>login authentication default</b></pre>
To test the authentication:
<pre>
SW1#<b>test aaa group radius admin P@ssw0rd legacy</b></pre>

# IEEE 802.1X
Port-based Network Access Control (PNAC)

**Scenario:** If a PC is connected to a switch first authenticate then bring the port up.

* 802.1X also involves three parties:
  * Authentication server: RADIUS
  * Authenticator: The Switch
  * Supplicant: The PC is going to be connected to the switch

For port-based authentication, both the switch and the end user’s PC must support the
802.1X standard, using the Extensible Authentication Protocol over LANs (EAPOL).

802.1X EAPOL is a Layer 2 protocol. 802.1X EAPOL is a Layer 2 protocol. At the point that a switch detects the presence
of a device on a port, the port remains in the unauthorized state. Therefore, the client
PC cannot communicate with anything other than the switch by using EAPOL. If the PC
does not already have an IP address, it cannot request one. The PC also has no knowledge
of the switch or its IP address, so any means other than a Layer 2 protocol is not possible.
This is why the PC must also have an 802.1X-capable application or client software.

<img src="https://user-images.githubusercontent.com/31813625/32982677-3fd7e308-cc56-11e7-9198-8cce2f610ee1.png" width="528" height="134" />

<pre>
SW1(config)#<b>aaa new-model</b>
SW1(config)#<b>aaa authentication dot1x default group radius</b>
SW1(config)#<b>dot1x system-auth-control</b>
SW1(config)#<b>interface range fastEthernet 3/0/1 - 24</b>
SW1(config-if-range)#<b>dot1x port-control ?</b>
  auto                PortState will be set to AUTO
  force-authorized    PortState set to Authorized
  force-unauthorized  PortState will be set to UnAuthorized
</pre>
* **Auto:** The port uses an 802.1X exchange to move from the unauthorized
to the authorized state, if successful. This requires an 802.1X-capable
application on the client PC.
* **force-authorized:** The port is forced to always authorize any connected
client. No authentication is necessary. This is the default state for all
switch ports when 802.1X is enabled.
* **force-unauthorized:** The port is forced to never authorize any connected
client. As a result, the port cannot move to the authorized state to pass
traffic to a connected client.

To allow multiple hosts on a switch port:
<pre>
SW1(config-if-range)#<b>dot1x host-mode multi-host</b></pre>

### Verification:
<pre>
SW1#<b>show dot1x all</b>
Sysauthcontrol              Enabled
Dot1x Protocol Version            2
Critical Recovery Delay         100
Critical EAPOL             Disabled

Dot1x Info for FastEthernet3/0/1
-----------------------------------
PAE                       = AUTHENTICATOR
PortControl               = AUTO
ControlDirection          = Both
HostMode                  = MULTI_HOST
ReAuthentication          = Disabled
QuietPeriod               = 60
ServerTimeout             = 30
SuppTimeout               = 30
ReAuthPeriod              = 3600 (Locally configured)
ReAuthMax                 = 2
MaxReq                    = 2
TxPeriod                  = 30
RateLimitPeriod           = 0

Dot1x Info for FastEthernet3/0/2
-----------------------------------
 --More--
</pre>