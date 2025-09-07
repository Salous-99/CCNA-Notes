# DHCP Snooping
DHCP snooping is a security feature that is used to filter out "bad" DHCP messages sent to untrusted ports on a switch. DHCP snooping only affects DHCP messages, not anything else. 
## Trusted and Untrusted Ports
See here:
![[Pasted image 20250906163127.png]]
Switches SW1 and SW2 route traffic to and from the end hosts, downlink ports should not be trusted, whilst uplink ports can be trusted. The network administrator cannot know for certain what the users on the end hosts will do as they do not control the devices directly nor the users, so ports receiving user traffic should not be trusted, and DHCP snooping should be enabled on them to prevent against a DHCP attack from the end devices. Uplink ports however can be trusted since they are connected to other network devices that are directly controlled by the administrator making them safe.

**Note:** generally, if the ports lead somewhere deeper into the network, they are downlink interfaces, and if they lead to the edges/periphery of the network, they are uplink interfaces
## DHCP Snooping Operations
What does DHCP snooping actually do though?? If a port receives a DHCP message, and it has DHCP snooping enabled, it will inspect the contents of the DHCP message to ensure that it is okay, dropping the message if it is not. If DHCP snooping is not enabled (port is trusted) the DHCP message is simply forwarded. How does it do this? When inspecting a DHCP message, the switch can figure out whether or not it is a client or server message by looking at the DHCP opcode field in the message, if it is 1, it is a client request message, if it is a 2, it is a server reply message. By default, all server DHCP messages ingress to untrusted ports are dropped immediately and with no further checks; this makes sense as untrusted ports are supposed to be connected to end hosts, and end hosts that are not network devices should not be sending DHCP server messages, it indicates that they are trying to imitate a legitimate server. DHCP client messages coming on untrusted ports are first examined before a decision is made about forwarding or dropping. The switch makes this determination based on message type. Here are the types of DHCP messages for client and server: 
- Server Messages:
	- OFFER
	- ACK
	- NACK
- Client Messages:
	- DISCOVER
	- REQUEST
	- RELEASE
	- DECLINE

The process works as follows:
- If receiving a server message on an untrusted port, drop message immediately.
- If receiving a client message on an untrusted port, inspect it:
	- If DHCP Discover/Request: check that the CHADDR and source MAC address match, if they don't, drop the packet
	- Release/Decline: check if the packets source IP address and receiving interface match the entry in the *DHCP Snooping Binding Table*, if they match, forward the packet, else, drop it. This ensures that only the host with the IP address lease can release its leased IP address
**Note:** New entries are added to the *DHCP Snooping Binding Table* when an IP address is successfully leased to an end host, the DHCP Snooping Binding Table contains the following fields:
- MAC Address of host (as per CHADDR)
- Leased IP address
- Time left before lease is up in seconds
- Type
- VLAN number
- Interface on switch connected to device
## DHCP Snooping Rate Limiting
DHCP snooping can also limit the rate at which DHCP messages are allowed to enter an interface. If the number of received messages crosses the configured threshold, the interface transitions to an err-disabled state and must be manually restarted using **shut** and **no shut** or automatically using **errdisable recovery cause *dhcp-rate-limit***.

Once configured with number N, it means that the configured interface can, at most, receive N DHCP messages **per second**. Exceeding that limit will disable the interface.
# DHCP Attacks
## DHCP Exhaustion Attack
Covered in previous section (see [[Port Security]]), but briefly:
- Attacker sends multiple DHCP discover requests with spoofed MAC addresses in order to reserve as many IP addresses as possible, preventing other users from accessing the network as they cannot be assigned an IP.
- If the DHCP server in the same LAN, the MAC address can simply be spoofed for the attack to work.
- If the DHCP server is not within the LAN, the CHADDR or *Client Hardware Address* can be spoofed for the same effect.
## DHCP Poisoning (Man-in-the-Middle)
Similar to ARP poisoning, a DHCP Poisoning attack is when the attacker sets up a fake DHCP server and responds to DHCP discover messages and answers with illegitimate information, the most common procedure is to insert the IP address of the attacker's computer as the default gateway; that way, hosts on the network will connect to the attacker first, allowing them to inspect and modify the packets contents before forwarding them to the default gateway.

This works because a host will accept the first DHCP offer it receives so at worst, it is a matter of time and some luck before the victim accepts the offer message by the attacker, the attacker in turn will assign the host a proper IP address, DNS server, and default gateway so the PC has no way of knowing it connected to an attacker.

Furthermore, if the DHCP server is not within the LAN, whilst the attacker's device is, the attacker's DHCP offer will almost certainly reach the victim before the legitimate DHCP offer. The host will respond with an ACK to the attacker and a "decline" message to the legitimate DHCP server after receiving its offer.
# Cisco IOS Configuration
To enable DHCP snooping, it must be done as part of a two step process: first, in global config mode, enable DHCP snooping globally using:
	**ip dhcp snooping**
Then enable it for a specific VLAN or VLANs using:  
	**ip dhcp snooping vlan <VLAN num\>**
Afterwards, run the command **\***: 
	**no ip dhcp snooping information option**	
Finally, run the command:
	**ip dhcp snooping trust**
Within the interface configuration mode for the trusted interface(s) to signal to the switch that these interfaces are trusted (all interfaces are untrusted by default)

**\*Note:** One of the many DHCP options is option 82 which is known as the *DHCP Relay Agent Information Option*. When added, additional information about the DHCP relay agent is provided. This is good sense if there is a remote DHCP server that needs to keep track of host information like:
- The VLAN is this host located in.
- The specific relay agent that forwarded the request to the server.
- The specific interface the DHCP request was received on
- etc...
By default, switchports with DHCP snooping enabled will add option 82 on DHCP messages they receive from the connected clients, even if the switch isn't acting as a relay agent, also by default, switches will drop DHCP messages with option 82 enabled if received on an untrusted port. If we refer to the earlier network:![[Pasted image 20250906163127.png]]
If PC1 were to send a discover request to R1, SW1 will, by default, add option 82 to the DHCP discover request and forward it to SW2, but since the connected interface on SW2 is not trusted, SW2 will drop the DHCP message before it gets to R1. As such, the feature to automatically add option 82 is disabled using the above command:  **no ip dhcp snooping information option**. Note that the same thing must be done on SW2 otherwise it (SW2) will add option 82 which will then cause R1 to drop the request as the message was not sent from a DHCP relay agent. 

To configure DHCP Snooping rate limits, run the comand:
	**ip dhcp snooping limit rate <messages per second\>**
From interface configuration mode.