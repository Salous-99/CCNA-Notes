![[Pasted image 20250626165204.png]]

Class D addresses are multicast addresses, more on those later. Class E addresses are reserved for experimental purposes, they are beyond the scope of the CCNA.

Class A addresses do not actually include the 127 (even though it is covered in the range) because the 127.X.X.X addresses are loop-back addresses, they are used to test the network stack of the local device. Packets sent to 127.X.X.X IP addresses will be looped back to the same device.

Furthermore, all networks have two reserved addresses, the network address and the broadcast address. The network is address is the one where all the rest/host bits (bits not in the mask) are zeros, and the broadcast address is the one where all the host bits are ones.

When assigning network addresses to all your different subnets, remember that the total number of addresses (that can be assigned to hosts) is equal to 2^(number of host bits) - 2, where the number of rest bits is equal to 32 - no. of mask bits and the -2 stems from the first and last addresses (all host bits equal to 0 or to 1) being the network and broadcast addresses respectively.

## Cisco IOS Commands
On a router, whilst in privileged EXEC mode, you can run
	**show ip interface brief** OR
	**sh ip int br** for short 
To list all the ports on a router and some other information about those interfaces:
- Interface name
- Assigned IP address
- The OK? column indicating whether or not the assigned IP address is valid (this is a legacy feature you can ignore)
- Method by which an interface was assigned an IP address
- Layer 1 status of the interface (connected on both ends with a cable? connected but not allowing traffic through, etc...)
- Protocol being used on layer 2 (Ethernet is common) 

To configure an interface, first log into global configuration mode and enter the command
	**interface \<interface name\>**
For example
	**interface gigbitethernet0/0** OR
	**in g0/0** for short
You can tell you are in interface configuration mode because the terminal will display: *deviceHostname*(config-**if**)#.

Once in interface configuration mode, enter the command
	**ip address \<ip address> \<subnet mask>**
For example, if we are configuring a router interface on the network 10.0.0.0/8 at the last usable address, we can enter the command:
	**ip address 10.255.255.254 255.0.0.0**

By default, the **shutdown** command has been issued for all interfaces on the router which makes their Status "administratively down" and their protocol "down". After configuring the IP addresses, issue the command
	**no shutdown** OR
	**no shut** for short
in order to enable the interface.

To check if everything was done correctly, you can check an interface or interfaces using:
	**show interfaces \<interface name>**

When configuring an interface, you can assign it a description as well which is just a string. To do so, enter interface configuration mode for the interface in question and issue the command:
	**description \<YOUR DESCRIPTION HERE>** OR
	**desc \<YOUR DESCRIPTION HERE>** for short

In privileged EXEC mode, you can issue the command
	**show interface description** OR
	**sh int desc** for short
For a brief overview of each interface as well as its description.