SSH!!! The tool for remote terminals. 10/10. favorite tool by far.
## Console Port Security
First off, we need to discuss console port security. Simply put, it is a good idea to add a password so that no one connected to the port can configure the terminal. To do that, first enter the command:
	**line console 0**
To enter console line configuration mode. Note that the number 0 is the index of the console line. Also note that there is only one singular console line per Cisco device which is line 0. This means that only one person can access the CLI at any given moment,

Then, in console line configuration mode, enter the command:
	**password <password\>**
Finally, enable password login by issuing the command:
	**login**
Now, when you connect to the console (via console port or by clicking on the console button in Packet Tracer) you will be prompted for a password.

You can also instantiate user profiles and require that any sign on to the device be a registered user that knows the specific username-password pair. To do this, in global configuration mode, enter the command:
	**username <username\> secret <password\>**
Then, enter console line configuration mode and enable password logins, the two commands are:
	**line console 0** to enter console line configuration mode
	**login local** 
The second command is similar to the plain "login" from before however it tells the device to allow only registered users to login and will prompt for a username and password.

A good security practice is to log out users after a brief period of inactivity. To set this timeout period use the command:
	**exec-timeout <minutes\> <seconds\>**
## Managed Switch
Switches are layer 2 devices that do not rely on, nor use IP addresses to forward frames to their destination. Layer 3 switches (multi-layer switches) can be assigned IP addresses on their port but that is a different story. A *managed switch* is a layer 2 device that is assigned an IP address so that it can be accessed remotely via **telnet** or **ssh** for configuration purposes.

To configure a layer 2 switch with an IP address, you can assign the IP address to a Switch Virtual Interface or SVI. Similarly to how you would bind an IP address to an interface for a router, you can do the following in a switch:
	**interface vlan <VLAN ID\>**
	**ip address <Switch IP address\> <subnet mask\>**
	**no shutdown** , sometimes this is not necessary as SVIs are enabled by default
You also need to configure the default gateway of this switch so that hosts outside of the VLAN can communicate to it. Consider the following network: 
![[Pasted image 20250824181140.png]]
You can bind the IP addresses .1.253 and .2.253 to the switches' SVIs however; PC1 will not be able to access SW2, and PC2 will not be able to access SW1.

To remedy this, we configure a default route for both switches. Since switches do not construct and populate a routing table, the route to the network must be hard coded. To do this, issue the command:
	**ip default-gatewat <IP address of local router/gateway interface\>**
This command can be issued on SW1 with the gateway address being 192.168.1.254 (router interface) this will allow control traffic (generated from telnet or ssh) to be routed over the network between PC2 and SW1.
## Telnet
Telnet was developed at 1969 in order to access a CLI of a remote host. Telnet has been replaced by ssh as telnet is not secure and sends everything in plaintext. Telnet servers listen on TCP/23.

In order to configure a network device as a telnet server, first set a password using:
	**enable secret <password\>**
You can also set a username and password for local login using
	**username <username\> secret <password\>**
Optionally, you can configure an ACL to limit the devices that can access the terminal remotely, to do this, run:
	**access-list <ACL number\> permit host <IP address of admin's PC\>**
Then to enter VTY line configuration mode, enter the command:
	**line vty 0 15**
Unlike the console line, there are 16 distinct VTY lines. The above command will enter VTY lines configuration mode for all 16 of them. From there, you can issue the:
	**login local**
command to enable user login, set the timeout period for inactivity using:
	**exec-timeout <minutes\> <seconds\>**
Afterwards, you can allow SSH and Telnet connections to be made via:
	**transport input {telnet | ssh | telnet ssh | all | none}**
and use the telnet flag. Finally, you can apply the optional ACL from earlier to the VTY lines using:
	**access-class <ACL number from earlier\> in**
"in" because we want to filter by source IP address. Note that applying an ACL to VTY lines uses yet another **access-class** command instead of the normal **ip access-group**.
## SSH
Two versions are currently available. Best to use version 2 instead of 1. If the device supports both, it is said to be running SSH 1.99 which simply means it supports both. SSH is like telnet but with strong encryption so that no one can read the data in plaintext if they happen to be sniffing packets. SSH servers listen for connections on TCP/22

To check if your device supports SSH, you can run the command:
	**show version**
This is used to display IOS information. To verify if your device supports SSH, look at the image name on the first line. If the name contains K9, the IOS version supports SSH. Some IOS images do not support strong forms of encryption (or don't support encryption at all), these images are called *No Payload Encryption* IOS images or NPE images. A more straightforward way to check is to run:
	**show ip ssh**

To enable SSH v2 on a device, you must first create an RSA key pair needed for the encryption to take place. To do that, first you must specify the domain name of a device using:
	**ip domain name <domain name\>**
This will set the domain name of the device which, in turn, will grant the device a Fully Qualified Domain Name or FQDN, an FQDN is required to generate RSA keys as a single RSA key is attached to an FQDN. After specifying the name, you can generate the key using:
	**crypto key generate rsa modulus <key length\>**
Alternatively, you can use
	**crypto key generate rsa**
And enter the length in the prompt you will receive. For SSH v2, RSA keys must be at least 768 bits long.

After generating an RSA key pair for the device, SSH v2 will be enabled by default. Then, in order to setup SSH v2, you go through a similar process as before:
	**enable secret <password\>**
	**username <username\> secret <password\>**
	**access-list <ACL number\> permit host <IP address\>** (optional)
	**ip ssh version 2** (this is the only extra command)
	**line vty 0 15** (enter vty line configuration mode)
	**login local**
	**exec-timeout <minutes\> <seconds\>**
	**transport input ssh**
	**access-class <ACL number\> in** (optional)

