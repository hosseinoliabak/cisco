# Recovering from a Lost or Forgotten Password (Catalyst 3750 SERIES)
1. Power off the standalone switch or the entire switch stack.
2. Reconnect the power cord to the standalone switch and, within 15
seconds, press the Mode button while the System LED is still flashing
green. Continue pressing the Mode button until the System LED turns
briefly amber and then solid green; then release the Mode button.

If you see this message proceed with Password Recovery Enabled below,
otherwise proceed with [here](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3750/software/release/12-2_25_sec/configuration/guide/3750scg/swtrbl.html#wp1090113)

<pre>
The password-recovery mechanism is <span style="font-weight: bold;background-color: #FFFF00">enabled</span>.

The system has been interrupted prior to initializing the
flash filesystem.  The following commands will initialize
the flash filesystem, and finish loading the operating
system software:

    flash_init
    load_helper
    boot


switch:
</pre>

3. Initialize the flash file system and display the contents of flash
memory:
<pre>
switch: <b>flash_init</b>
Initializing Flash...
flashfs[0]: 4 files, 1 directories
flashfs[0]: 0 orphaned files, 0 orphaned directories
flashfs[0]: Total bytes: 15998976
flashfs[0]: Bytes used: 8659968
flashfs[0]: Bytes available: 7339008
flashfs[0]: flashfs fsck took 9 seconds.
...done Initializing Flash.
Boot Sector Filesystem (bs) installed, fsid: 3
Setting console baud rate to 9600...
switch: <b>dir flash:</b>
Directory of flash:/

2    -rwx  8625186   <date>               c3750-ipservicesk9-mz.122-35.SE2.bin
3    -rwx  29769     <date>               config.text
4    -rwx  676       <date>               vlan.dat
5    -rwx  1935      <date>               private-config.text

7339008 bytes available (8659968 bytes used)
</pre>

4. Rename the configuration file to something like `config.text.old`.
This file contains the password definition. Then boot the system
<pre>
switch: <b>rename flash:config.text flash:config.text.old</b>
switch: <b>boot</b>
</pre>

5. When the switch is booted up go to privilege mode and rename the
configuration file to its original name:
<pre>
Switch# <b>rename flash:config.text.old flash:config.text</b>
</pre>

Note Before continuing to Step 9, power on any connected stack members
and wait until they have completely initialized. Failure to follow this step can result in a lost configuration depending on how your switch is set up.

6. Copy the configuration file into memory:
<pre>
Switch# <b>copy flash:config.text system:running-config</b>
Destination filename [running-config]?
</pre>

Press Return in response to the confirmation prompts.
The configuration file is now reloaded, and you can change the password.

7. Enter global configuration mode and change the password:
<pre>
HMLADMP01C01# configure terminal
HMLADMP01C01(config)# enable secret password
HMLADMP01C01(config)#username test privilege 15 password test
</pre>

8. Write the running configuration to the startup configuration file in
privilege mode:
<pre>
HMLADMP01C01(config)#end
HMLADMP01C01#copy running-config startup-config
</pre>
The new password is now in the startup configuration.