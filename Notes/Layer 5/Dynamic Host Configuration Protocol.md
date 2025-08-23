DHCP allows for the automatic configuration of the following things on an end-host:
- IP address
- Subnet mask
- DNS server
- Default gateway's address
- Default domain name
Manual configuration can be done for small enough networks but larger networks need something automated to account for scalability. Certain devices within the network will probably still need manual configuration; devices like routers, servers and other layer 3 or above network devices, these devices should have a fixed address as they serve a function for the rest of the devices on the network.

A DHCP server is usually running on your plug-and-play router, but for larger networks with more devices and where the routers are not plug-and-play, a separate windows or linux server must be set up that acts as the DHCP server.
## IP Address Assignment
When a computer first enters the network and wants to connect, it needs to know certain things about the network in order to communicate, chiefly, its own IP address and subnet mask. If it knows that much, it can contact all devices within the local LAN (including default gateway which should route packets to the internet). However, how does it do that with on IP address first being initially and manually configured? 

The DHCP process for attaining an IP address goes like this:
1. The DHCP client (end host/your PC or laptop) will send a **Discover** DHCP message to the broadcast address (for layer 2 and 3). The purpose of this message is to find any DHCP servers within the broadcast domain. The reason as to why the message is broadcast is because the device does not know the IP address of the server, the device doesn't necessarily know that there is a DHCP server in the LAN. 
	1. The message itself says: "Where is the DHCP server? Can it please assign me an IP address?"
	2. The destination IP and MAC addresses are set to broadcast
	3. The source MAC address is present but the IP address is set to 0.0.0.0
	4. The source port is UDP/68 (always UDP 68 for client side DHCP)
2. The DHCP server(s) replies to the message with either a broadcast or unicast **Offer** message. The purpose of this message is to offer a specific IP address to the device 
	1. The message itself says: "You can have this IP address if you want"
	2. The destination IP and MAC addresses could be set to broadcast or unicast depending on what was requested in the discover message. If the frames are unicast then the IP address used for the destination will be the same IP in the offer message
	3. The source MAC and IP addresses are both present
	4. The source port is UDP/67 (always UDP 67 for server side DHCP)
3. The DHCP client replies with broadcast **Request** message, in order to confirm that it wants the IP address that it was offered.
	1. The message says: "Yes please, I will have the IP address you offered me"
	2. The destination IP and MAC address are set to ***broadcast\****
	3. The source MAC address is present but the IP could either be 0.0.0.0 or the IP offered in the previous message. It depends on what the client does.
4. The DHCP server replies with an **Ack** message, finalizing the process.
	1. The message says: "here you go"
	2. The destination MAC and IP addresses are both known now and the frame sent is a unicast frame.
5. **Not a part of the process, just related:** There is a **release** message that the device currently leasing an IP from a DHCP server can send, as a unicast frame, to the server to tell it that the leased IP is no longer being used and can be returned to the pool of available IP addresses.

**\*Note:** The reason as to why the request message is sent at all, and more importantly, why it is sent as a broadcast is because there could be multiple DHCP servers on the LAN that all respond to any one discover message, and all are allowed to respond. The DHCP request message signals to the chosen DHCP server (usually the first one to reply to a discover message is chosen) that it is using its IP address and not any other, it also signals to the other DHCP servers that their IP offers are not being accepted and that they can be used for other devices.
## DHCP Relay Agent
Typically, for home networks and small enterprises, it is enough to have the router act as a DHCP agent for the connected LANs. However, for larger networks, it could be useful to centralize the DHCP process into one server. Doing so however will pose some challenges as broadcast messages are not forwarded by routers. That means if the central DHCP server is on another subnet from the devices (which will definitely be the case), messages won't reach it.

This is where DHCP relay agents come in, a router can be set up as a DHCP relay agent which allows it to listen for DHCP discover messages on the connected subnet(s) and forward those messages to the DHCP server, as well as relay the responses back to the device that initiated the request. So...it acts as a relay for DHCP messages. Not revolutionary I suppose.

# Cisco IOS Configuration
## DHCP Server
When configuring a DHCP server, first you configure all the relevant data that will be used in the messages, and reserve a pool of IP addresses that can be assigned to the connected hosts. To do that, in global configuration mode, run:
	**ip dhcp excluded-addresses <first address\> \[last address\]**
	**ip dhcp pool <IP address pool's name\>**
The first command excludes certain IP addresses from the pool, **note that the second address is optional because you can ban just one address if you need to.** The excluded addresses can be reserved for static assignment of IP addresses to critical devices (like a server or any given network device). the second command creates a DHCP pool with the given name and enters you into DHCP configuration mode. From there, you can assign the IP addresses to the pool using the command:
	**network <network address\> \<subnet mask IP or CIDR notation\>**
This **network** command, unlike previous instances, does not activate DHCP on any interfaces, it simply tells the router that it can pull addresses from the network provided (except for the reserved addresses) and assign them to hosts that request them. After this critical step, you can assign the other information:
	**dns-server <IP of DNS server\>** assigns DNS server
	**domain-name <domain name\>** assigns default domain
	**default-router** <IP of default gateway\>
	**lease <days\> <hours\> <minutes\>** assigns a specific lease time OR
	**lease infinite** assigns a lease indefinitely (not recommended)

**\*Note:** the host will try to renew the least before it expires, if it can't it will hold onto it until it expires and then simply release it.

To show the assigned IP addresses run:
	**show ip dhcp binding**
## DHCP Relay Agent
Doing this is simple, you only need one command; go to interface configuration mode of the interface connected to the subnet that will be required to listen for incoming DHCP discover messages and issue the command:
	**ip helper-address <IP address of DHCP server\>**
This assumes that this router has a route to the server, if not, configure one statically or dynamically.
## DHCP Client
This is rare to do as servers, routers and other network devices are usually configured with static addresses but a router can still receive an IP address from a DHCP server. To do that, navigate to the interface configuration mode of the interface lacking an IP address and run the command:
	**ip address dhcp**
that it. This -of course- assumes that the interface is connected to a subnet that contains a DHCP server or is connected to another router's interface that is configured as a DHCP relay agent.

**\*Note:** The last two sections show us that DHCP works on routers on a per-interface basis.