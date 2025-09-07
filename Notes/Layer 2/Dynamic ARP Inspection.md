Dynamic ARP Inspection or DAI is to ARP what [[DHCP Snooping]] is to DHCP. It is a layer 2 *protocol-like* scheme to ensure that no useless or malicious ARP requests travel across the network.
# Dynamic ARP Inspection
Similar to DHCP snooping, DAI affects only ARP messages, and nothing else, and it only inspects ARP messages received on untrusted interfaces. All ports are untrusted by default, usually when using DAI, interfaces connected to end hosts are not trusted and interfaces connected to other network devices should be trusted. You can still configure downlink interfaces as untrusted even if they are connected to another network device, but the Cisco documentation says to not do that, but both configurations will work fine.
## DAI Operations
DAI inspects incoming ARP messages on its untrusted ports and checks their validity in one of two ways:
- Checks the DHCP snooping binding table to ensure that the sender's IP and MAC addresses match the corresponding entry in the binding table. If they do match, the packet is forwarded, else it is dropped.
- Uses a preconfigured ACL to validate correct ARP messages.
ARP ACLs can be used to map IP and MAC addresses for DAI to check. It is useful for hosts that do not use DHCP, thus the switches wouldn't necessarily have DHCP snooping binding tables to check. DAI can be configured to perform more in-depth checks but these are optional.

DAI also supports rate-limiting to prevent an attacker from overwhelming the switch with ARP messages.

**Note:** both DAI as well as DHCP snooping require processing power and that processing is done on the devices' CPUs. This means that an attacker can still overwhelm the processors of the network devices, even if the end packets themselves are dropped, placing an interface in the err-disabled state however will do the trick.
## ARP ACLs
It is likely that this concept is not within the scope of the CCNA. However, just a brief mention of how it works and some configuration details won't hurt.

Assuming that a certain network does not rely on DHCP, its switches will not have DHCP Snooping binding tables, a more common situation would involve a network that does utilize DHCP and its switches do have DHCP binding tables however, a server (end host) on the network would probably have a statically assigned IP address and would not utilize DHCP, and thus would not have an entry in the binding table. If it does not have an entry in the binding table, all of its ARP messages will be dropped because its connected interface is untrusted. You could trust the interface to prevent this from becoming a problem or you could configure an ACL to allow the server to send ARP messages. To configure an ACL, you use the command:
	**arp access-list <ACL name\>**
This will move you to ARP ACL configuration mode; to create an entry, you issue the command:
	**permit ip host <IP address of server/end host\> mac host <MAC address of server/host\>**
After making the ACL, you can apply it to an interface fro global configuration mode using:
	**ip arp inspection filter <ACL name from before\> vlan <VLAN number\>**
# ARP Related Attacks
## ARP Poisoning
Covered earlier in [[Security Fundamentals]], ARP poisoning involves an attacker impersonating another host and replying to ARP requests meant for that host so that the victim connects to the attacker's computer instead of the intended destination, effectively making the attacker's computer a man in the middle.

The attacker can also send a gratuitous ARP with the victim's IP address to impersonate them directly before any host within the LAN sends an ARP request for that IP.
# Cisco IOS Configuration
You can enable DAI in global configuration mode per VLAN using:
	**ip arp inspection vlan <VLAN number\>**
In order to define a trusted interface, navigate to its interface configuration mode and then issue the command:
	**ip arp inspection trust**
To view all port status with respect to DAI, use:
	**show ip arp inspection interfaces**
This will show 4 columns:
1. Interface
2. Trust state: trusted or not trusted in the DAI process
3. Rate
4. Burst Interval
About those last two: rate limiting in DAI, unlike in DHCP snooping which has only one rate value 'N' that defines the upper limit to be N messages per second, DAI uses a **burst interval** system that allows N messages to be sent in M seconds. "Rate" is the number N, and "Burst Interval" is the number M. **Note:** by default, rate limiting is disabled on all trusted interfaces and enabled on all untrusted interfaces. To configure rate limiting, from interface configuration mode, issue the command:
	**ip arp inspection limit rate <Number of messages\> burst interval <Interval in seconds\>**
If the *burst interval* option isn't specified, it will use the default 1 second interval time. If an interface receives a number of ARP messages that exceeds the maximum allowed rate, the interface becomes error disable and can be re-enabled manually via the traditional **shut** into **no shut** or automatically using:
	**errdisable recovery cause *arp-inspection***

For other optional checks that DAI can perform, there is:
- **Source MAC:** checks to see that the source MAC address in the layer 2 header matches the sender MAC address in the DHCP message body. 
- **IP**: Ensures that the destination IP address is not unexpected, i.e. it ensures that it isn't a network or broadcast address, it ensures that it isn't a well known multicast address. The device checks the sender IP in all ARP requests and responses, but it checks the destination IP only in responses. 
- **Destination MAC**: checks to see that the destination MAC address in the layer 2 header matches the target MAC address in the DHCP message

To enable these checks, you can issue the command:
	**ip arp inspection validate {dst-mac | ip | src-mac}**

Three things to note:
1. If you issue the command multiple times to enable multiple checks, only the latest command (and therefore check) will apply.
2. To enable multiple checks, you need to enter the command:
	   **ip arp inspection validate <list of checks, space separated\>**
   e.g.
	   **ip arp inspection validate src-mac dst-mac**
3. These are **additional** checks that will be performed alongside the initial check that relies on the DHCP binding table or ACL, it does not replace that original check.

Finally a nice show command to tie it all together:
	**show ip arp inspection**
Shows you detailed information about the DAI process running on the switch.