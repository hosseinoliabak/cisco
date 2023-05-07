# Introduction to NX-OS Command Line

In this section, we assume that you know how to connect to a NX-OS CLI via a terminal emulator program. Here is how we bootstrap our Nexus switch for the first time.

If you **skip** Power On Auto Provisioning, you must configure the password later, you also must tell the switch with global configuration command of `boot nxos64-cs.10.2.2.F.bin` where to boot the NX-OS from. Failing to do so, would result in loosing the configuration and starting from boot loader after the reboot.

<pre>
2022 Apr 22 23:49:35 switch %$ VDC-1 %$ %POAP-2-POAP_INFO: - Abort Power On Auto Provisioning [yes - continue with normal setup, skip - bypass password and basic configuration, no - continue with Power On Auto Provisioning] <b>(yes/skip/no)</b>[no]:
<b></u>skip</u></b>

!!! NOTE: You have selected skip option. POAP will be aborted and password configuration will be skipped !!!
Disabling POAP.......Disabling POAP
2022 Apr 22 23:49:54 switch %$ VDC-1 %$ poap: Rolling back, please wait... (This may take 5-15 minutes)
Disabling lldp

2022 Apr 22 23:50:36 switch %$ VDC-1 %$ %COPP-2-COPP_POLICY: Control-Plane is protected with policy copp-system-p-policy-strict.



User Access Verification
 login: admin
Password: {No Password Was Entered Here}
 
Cisco NX-OS Software
Copyright (c) 2002-2021, Cisco Systems, Inc. All rights reserved.
Nexus 9000v software ("Nexus 9000v Software") and related documentation,
files or other reference materials ("Documentation") are
the proprietary property and confidential information of Cisco
Systems, Inc. ("Cisco") and are protected, without limitation,
pursuant to United States and International copyright and trademark
laws in the applicable jurisdiction which provide civil and criminal
penalties for copying or distribution without Cisco's authorization.

Any use or disclosure, in whole or in part, of the Nexus 9000v Software
or Documentation to any third party for any purposes is expressly
prohibited except as otherwise authorized by Cisco in writing.
The copyrights to certain works contained herein are owned by other
third parties and are used and distributed under license. Some parts
of this software may be covered under the GNU Public License or the
GNU Lesser General Public License. A copy of each such license is
available at
http://www.gnu.org/licenses/gpl.html and
http://www.gnu.org/licenses/lgpl.html
***************************************************************************
*  Nexus 9000v is strictly limited to use for evaluation, demonstration   *
*  and NX-OS education. Any use or disclosure, in whole or in part of     *
*  the Nexus 9000v Software or Documentation to any third party for any   *
*  purposes is expressly prohibited except as otherwise authorized by     *
*  Cisco in writing.                                                      *
***************************************************************************
switch# {Here you have jumped to privilege EXEC mode. No User Mode Like with Catalyst}
</pre>

Upon typing yes Power On Auto Provisioning, you will continue with normal setup starting by setting up a password. (This is recommended)

```
2022 Apr 23 00:38:43 switch %$ VDC-1 %$ %POAP-2-POAP_INFO: - Abort Power On Auto Provisioning [yes - continue with normal setup, skip - bypass password and basic configuration, no - continue with Power On Auto Provisioning] (yes/skip/no)[no]: yes

Disabling POAP...                                                                                                                                                                                                    2022 Apr 23 00:38:56 switch %$ VDC-1 %$ poap: Rolling back, please wait... (This may take 5-15 minutes)
Disabling lldp


         ---- System Admin Account Setup ----


Do you want to enforce secure password standard (yes/no) [y]:

  Enter the password for "admin":
  Confirm the password for "admin":

         ---- Basic System Configuration Dialog VDC: 1 ----

This setup utility will guide you through the basic configuration of
the system. Setup configures only enough connectivity for management
of the system.

Please register Cisco Nexus9000 Family devices promptly with your
supplier. Failure to register may affect response times for initial
service calls. Nexus9000 devices must be registered to receive
entitled support services.

Press Enter at anytime to skip a dialog. Use ctrl-c at anytime
to skip the remaining dialogs.

 Would you like to enter the basic configuration dialog (yes/no): no

2022 Apr 23 00:39:52 switch %$ VDC-1 %$ %COPP-2-COPP_POLICY: Control-Plane is protected with policy copp-system-p-policy-strict.




User Access Verification
 login: admin
Password:

Cisco NX-OS Software
Copyright (c) 2002-2021, Cisco Systems, Inc. All rights reserved.
Nexus 9000v software ("Nexus 9000v Software") and related documentation,
files or other reference materials ("Documentation") are
the proprietary property and confidential information of Cisco
Systems, Inc. ("Cisco") and are protected, without limitation,
pursuant to United States and International copyright and trademark
laws in the applicable jurisdiction which provide civil and criminal
penalties for copying or distribution without Cisco's authorization.

Any use or disclosure, in whole or in part, of the Nexus 9000v Software
or Documentation to any third party for any purposes is expressly
prohibited except as otherwise authorized by Cisco in writing.
The copyrights to certain works contained herein are owned by other
third parties and are used and distributed under license. Some parts
of this software may be covered under the GNU Public License or the
GNU Lesser General Public License. A copy of each such license is
available at
http://www.gnu.org/licenses/gpl.html and
http://www.gnu.org/licenses/lgpl.html
***************************************************************************
*  Nexus 9000v is strictly limited to use for evaluation, demonstration   *
*  and NX-OS education. Any use or disclosure, in whole or in part of     *
*  the Nexus 9000v Software or Documentation to any third party for any   *
*  purposes is expressly prohibited except as otherwise authorized by     *
*  Cisco in writing.                                                      *
***************************************************************************
switch#
```

Choosing **no** or if you hit return key you have noed the prompt. In this case, you will stick in the Power On Auto Provisioning asking the same question.

### How to Navigate to Configuration Mode

Once you are logged in, you will see the Privilege EXEC mode command prompt. In this mode you can run any troubleshooting and verification commands. In this mode you cannot configure the NX-OS. You must escalate to configuration mode with `configure` command.

### Configuration Mode and Subconfiguration Modes

**Global configuration mode** is used to access configuration options on a Nexus switch.

**Line configuration mode** is used to configure console and SSH.

**Interface configuration mode** is used to configure a switchport.

### Navigation between NX-OS Modes

To move in to global configuration mode, use `configure` command in the privilege EXEC mode. To return to privilege EXEC mode, use the exit command continuously until you get there. You could alternatively hit <kbd>Ctrl</kbd> + <kbd>Z</kbd> or use `end` command. from global configuration or any subconfiguration mode to jump into the privilege mode.

### Hot Keys and Shortcuts

| Keystroke      | Description                                                           |
| -------------- | --------------------------------------------------------------------- |
| Tab            | Completes a partial command name entry                                |
| Backspace      | Erases the character to the left                                      |
| Ctrl + A       | Move the curser to the beginning if the line                          |
| Ctrl +E        | Move the cursor to the end of the line                                |
| Ctrl + U       | Delete the line from the cursor position to the beginning of the line |
| Ctrl + K       | Delete the line from the cursor position to the end of the line       |
| ↑ (Up Arrow)   | Brings the last command you issued (Moves backward in the history)    |
| ↓ (Down Arrow) | Moves forward to the history                                          |

*Table 1: Hot Keys and Shortcuts*

## Know your Chassis

To check the chassis host name and base MAC address use `show sprom backplane` followed by supervisor number command :

```
N9K01# show sprom backplane 1
DISPLAY backplane sprom contents:
Common block:
 Block Signature : 0xABAB
 Block Version   : 3
 Block Length    : 160
 Block Checksum  : 0x791
 EEPROM Size     : 0
 Block Count     : 5
 FRU Major Type  : 0x0
 FRU Minor Type  : 0x0
 OEM String      :
 Product Number  : N9K-C9300v
 Serial Number   : 9GRZKIVE65I
 Part Number     :
 Part Revision   :
 Mfg Deviation   :
 H/W Version     : 0.0
 Mfg Bits        : 0
 Engineer Use    : 0
 snmpOID         : 9.12.3.1.3.1510.0.0
 Power Consump   : 0
 RMA Code        : 0-0-0-0
 CLEI Code       :
 VID             :
Chassis specific block:
 Block Signature : 0x6001
 Block Version   : 3
 Block Length   : 39
 Block Checksum  : 0x15D
 Feature Bits    : 0x0
 HW Changes Bits : 0x0
 Stackmib OID    : 0
 MAC Addresses   : 50-00-00-02-00-00
 Number of MACs  : 128
 OEM Enterprise  : 0
 OEM MIB Offset  : 0
 MAX Connector Power: 0
WWN software-module specific block:
 Block Signature : 0x6005
 Block Version   : 1
 Block Length    : 0
 Block Checksum  : 0x66
 wwn usage bits:
 00 00 00 00 00 00 00 00
 00 00
License software-module specific block:
 Block Signature : 0x6006
 Block Version   : 1
 Block Length    : 16
 Block Checksum  : 0x77
 lic usage bits:
 00 00 00 00 00 00 00 00
 ```

To check the NX-OS Software Version along with memory and disk information as well as the system uptime use `show version` command:

```
N9K01(config)# show version
Cisco Nexus Operating System (NX-OS) Software
TAC support: http://www.cisco.com/tac
Documents: http://www.cisco.com/en/US/products/ps9372/tsd_products_support_serie
s_home.html
Copyright (c) 2002-2021, Cisco Systems, Inc. All rights reserved.
The copyrights to certain works contained herein are owned by
other third parties and are used and distributed under license.
Some parts of this software are covered under the GNU Public
License. A copy of the license is available at
http://www.gnu.org/licenses/gpl.html.

Nexus 9000v is a demo version of the Nexus Operating System

Software
  BIOS: version
  NXOS: version 10.2(2) [Feature Release]
  BIOS compile time:
  NXOS image file is: bootflash:///nxos64-cs.10.2.2.F.bin
  NXOS compile time:  12/14/2021 23:00:00 [12/15/2021 11:59:34]

Hardware
  cisco Nexus9000 C9300v Chassis
   with 16399552 kB of memory.
  Processor Board ID 90HFD7LJVD4
  Device name: N9K01
  bootflash:    4287040 kB

Kernel uptime is 1 day(s), 1 hour(s), 14 minute(s), 14 second(s)

Last reset
  Reason: Unknown
  System version:
  Service:

plugin
  Core Plugin, Ethernet Plugin

Active Package(s):
```

### Features

With NX-OS software we have the ability to enable and disable specific features, such as Interface-vlan, OSPF. To verify which features are already enabled:

```
N9K01(config)# show feature | include enable
icam                   1          enabled
license-smart          1          enabled
sshServer              1          enabled
```

To enable a feature type the command `feature FEATURE-NAME`:

```
N9K01(config)# feature ?
  analytics               Enable/Disable Analytics!!!
  bash-shell              Enable/Disable bash-shell
  bfd                     Bfd
  bgp                     Enable/Disable Border Gateway Protocol (BGP)
  container-tracker       Enable/Disable NXOS Container Tracker
  dhcp                    Enable/Disable DHCP Manager
  dot1x                   Enable/Disable dot1x
...
```

### Some handy commands while working with CLI

##### What is your present working context

The command where returns what your present working context is

```
N9K01(config)# where

  conf      admin@N9K01%default
N9K01(config)# interface ethernet 1/1
N9K01(config-if)# where

  conf; interface Ethernet1/1      admin@N9K01%default
N9K01(config-if)#
```

##### Negate a command using no

If you want to revert a configuration, you would use command no in the beginning of the regular command:

```
N9K01(config)# ip route 0.0.0.0/0 192.168.0.1
N9K01(config)# no ip route 0.0.0.0/0 192.168.0.1
```

##### Alias

You might be wanting using aliases for frequently used commands.

```
N9K01(config)# wr

                 ^
% Incomplete command at '^' marker.
N9K01(config)# cli alias name wr copy running startup
N9K01(config)# wr

2022 Apr 23 01:34:41 N9K01
[########################################] 100%
```

##### Run show commands from CLI

Unlike IOS or IOS-XE, you don’t need to run privilege EXEC command by adding do in the beginning of the command. You can run privilege EXEC commands directly from configuration mode:

```
N9K01(config-if)# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Eth1/1, Eth1/2, Eth1/3, Eth1/4
                                                Eth1/5, Eth1/6, Eth1/7, Eth1/8
CLI History
You can access the CLI history for the current session

N9K01(config)# show cli history unformatted | last 7
    ip route 0.0.0.0/0 192.168.0.1
    no ip route 0.0.0.0/0 192.168.0.1
    interface ethernet 1/1
      shutdown
      no shutdown
    exit
  show cli history unformatted | last 7
```

##### Issue multiple commands in a single line

You can separate multiple commands to the CLI using `;` .

```
N9K01# configure terminal ; interface ethernet 1/1 ; shutdown ; no shutdown
```

##### Toggling Between Different Command Modes

You can toggle between different configuration mode using `push` and `pop`

```
N9K01(config)# interface ethernet 1/1
N9K01(config-if)# push
N9K01(config-if)# end
N9K01# pop
Enter configuration commands, one per line. End with CNTL/Z.
N9K01(config-if)# where
  conf; interface Ethernet1/1      admin@N9K01%default
```

##### Checkpoint and Rollback

In order you take a snapshot of the current startup config you can use `checkpoint` global configuration command along with choosing a snapshot name. To restore the configuration, you would use the `rollback` global command.

```
N9K01(config)# show running-config interface ethernet 1/1

!Command: show running-config interface Ethernet1/1
!Running configuration last done at: Sat Apr 23 01:34:36 2022
!Time: Sat Apr 23 01:46:16 2022

version 10.2(2) Bios:version

interface Ethernet1/1

! No Configuration Here
N9K01(config)#
N9K01(config)# checkpoint CHK01
.Done
! Let's do some arbitrary configuration changes
N9K01(config)# interface ethernet 1/1
N9K01(config-if)# switchport access vlan 100
N9K01(config-if)# show running-config interface ethernet 1/1

!Command: show running-config interface Ethernet1/1
!Running configuration last done at: Sat Apr 23 01:47:39 2022
!Time: Sat Apr 23 01:47:52 2022

version 10.2(2) Bios:version

interface Ethernet1/1
  switchport access vlan 100


! Let's restore our last snapshot to  the running configuration
N9K01(config-if)# rollback running-config checkpoint CHK01
ADVISORY: Rollback operation started...
Modifying running configuration from another VSH terminal in parallel
is not recommended, as this may lead to Rollback failure.

Collecting Running-Config
Generating Rollback patch for switch profile
Rollback Patch is Empty
Collecting Running-Config
Generating Rollback Patch
Executing Rollback Patch
During CR operation,will retain L3 configuration
when vrf member change on interface
Generating Running-config for verification
Generating Rollback Patch

Rollback completed successfully.


! Let's verify if our configuration has been restored
N9K01(config-if)# show running-config interface ethernet 1/1

!Command: show running-config interface Ethernet1/1
!Running configuration last done at: Sat Apr 23 01:48:34 2022
!Time: Sat Apr 23 01:48:51 2022

version 10.2(2) Bios:version

interface Ethernet1/1

! Nothing Here
N9K01(config-if)#
```

##### Terminal Length 0 (Turn off Pagination)

Some commands have long output. By default you the output comes in pages in the terminal standard output. In order to see the whole command output outright, you need to turn of the pagination. Try this:

```
N9K01(config-if)# show running-config | no-more
```