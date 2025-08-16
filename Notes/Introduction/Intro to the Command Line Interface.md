Cisco network devices use the Cisco IOS, their own proprietary OS. In order to configure a Cisco device, you have to connect to it either via a web page if it's a modern device or simply through connecting to the console port present on the devices. If you are configuring the device for the first time, you will need to connect via console port.

Console ports on a Cisco switch:
![[Pasted image 20250625105045.png]]
port**s** plural because the RJ-45 connector to the side of the mini-B USB can also be used to connect. To connect, to the RJ-45, you don't use a standard Ethernet cable, you will instead use a ***Rollover cable*** which looks like this:
![[Pasted image 20250625105244.png]]
 The other end is called a DB-9 connector. Since most modern laptops don't have a serial port, you would probably need an adapter to convert to USB or type C.

The connections within the cable are:
![[Pasted image 20250625105421.png]]

Once connected, you have to use PuTTy or another terminal emulator to access the Cisco IOS CLI. Once on the CLI, you will log in *user-mode* sometimes also called *user EXEC mode* as indicated by the '>' sign following the host name. This operations that can be performed in user-mode are limited; you can view certain configurations but cannot change anything.

To leave user mode and into *privileged EXEC mode* you use the command 
	**enable**
Once in privileged EXEC mode, the '>' sign will change into a pound '#' sign. In privileged EXEC mode, you ave complete access to view the device configuration, restart the device and others, you still cannot change the configuration of the device however, you can change the time, save a configuration file, and other limited operations.

To actually configure the device, you need to be in *Global configuration mode* and to enter that mode, you have to use the command
	**configure terminal**
in the privileged EXEC mode. Once in this mode, the '#' sign will change into: "(config) #". Within this mode, you can make configuration changes to the device.

First thing you do is set a password so that others cannot configure the device if they had physical access to it. To do so, the command is:
	**enable password \<password goes here\>**
After setting the password, it is also useful to run, in global configuration mode, the command:
	**service password-encryption**
This command will encrypt the password stored in the configuration file so that it isn't visible in plaintext when printing the configuration file to terminal.

**Note:** subsequent passwords configured on the device will also be encrypted, i.e. if you run **enable password XXX** again, the new password stored will be stored in its encrypted format, not in plaintext. To disable this feature, you can run the command:
	**no service password-encryption**
This will **not** decrypt currently encrypted passwords, but it will disable encryption for future passwords set on the system.

Whilst this password is encrypted, the encryption algorithm it uses is a proprietary Cisco algorithm (referred to as "7"? i'm not sure) that provides relatively weak security. As such, a better method for setting passwords (instead of using enable password) is to run:
	**enable secret \<password goes here\>**
This method uses MD5 instead of the 7 algorithm.

**Note:** it is mentioned in the instructional videos that the password is encrypted, although it is likely that the password is hashed, not encrypted. This is based on the fact that no key was provided (although there could be some encryption key hard-wired into the device's ROM) also because the other algorithm used as opposed to this "7" algorithm is MD5 which is a hashing algorithm.

**Note:** The **service password-encryption** command has no effect on the **enable secret** command, whether or not password encryption is enabled, the **enabled secret** command will always provide an encrypted/hashed password, which is another reason as to why it is better to use.

**Note:** If you configured a password using both methods (enable password and enable secret), the password set using enable secret will be the one that is used since it has a higher level of encryption.

The configuration files for Cisco devices are split into 2, a *"running config"* and a *"start-up config"*. A the names imply, the running config is the one currently active, and the start-up config file is the one that will be loaded into memory in place of the running-config upon restart of the device. Adjustments made to the running configuration file (which is what is done in global configuration mode) will not carry over between restarts, you must make adjustments to the start-up configuration file for persistence, more on this later.

**Note:** upon initial setup of the device, the startup configuration would not be present and in its place, a default configuration would be loaded into memory instead.

To show the contents of a config file, simply run the command
	**show running-config** OR **show startup-config**

To write to the start-up config, you can run any of the following in **privileged EXEC mode**:
	**write**
	**write memory**
	**copy running-config startup-config**

All of the above perform the same function.

