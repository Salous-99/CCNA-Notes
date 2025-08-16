Why use IPv6? IPv4 works perfectly fine and we are used to it. Well, because there aren't enough IPv4 addresses to meet all of our demands. Every connected end host needs an IP address and we simply have more network interfaces than that, be they virtual or not. IPv6 has a much larger address space.

The solutions we have tried before are:
- VLSM: increases usage efficiency of private address spaces
- Private address spaces: reserving IP address ranges for private usage that are cannot be routed over the internet. This allows duplicate IP addresses in different LANs.

These however are short term solutions since we will eventually exhaust the space. That's why transitioning to IPv6 is a better idea.

IANA distributes the IPv4 address space to various Regional Internet Registries which assign them to regional companies that need them. These registries however have exhausted or are close to exhausting their IPv4 addresses.

IPv6 solves that problem by brute force: more bits!! an IPv6 address has way more than 32 bits like IPv4, in fact it has 128. This means the size of the IPv6 address space is 2^96 times larger than that of IPv4. 2^128 is a ridiculously large number. If we fully transition to IPv6, address exhaustion will not be a problem.

IPv6 addresses are written as 32 hexadecimal characters, each representing 4 bits of the address. Also, in IPv6, you can write the subnet mask using CIDR notation, even when configuring IPv6 on Cisco devices. IPv6 addresses can be abbreviated in the following ways:
1. removing lead zeros from each quartet
2. consecutive quartets of all zeros can be replaced with a double colon 
	1. can only be done once to one group of quartets of consecutive zeros
	2. Other not abbreviated zero quartets are shortened to 0

Breaking down an IPv6 address, like this one: 
	2001:0DB8:0B00:0001:0000:0000:0000:0001/64

- The first 3 quartets (2001:0DB8:0B00) are a 48-bit ***global routing prefix*** assigned by the ISP. These differ depending on what kind of IPv6 address it is, the above address is a global unicast address, other types will be mentioned later on
- Next quartet (0001) is a ***subnet identifier*** used by an enterprise to define different subnets
- The last 4 quartets (0000:0000:0000:0001) are the ***interface identifier*** used for the host portion of the address.

Typically ISPs hand out /48 blocks of IPv6 addresses to enterprises, and also typically, IPv6 subnets use a /64 mask. This means a company receiving a /48 block will use 16 bits (1 quartet) to identify a subnet, and the remaining bits to identify a host within a subnet.
# IPv6 Address Types
## EUI-64 Addresses
The first kind of address we will learn about is an Extended Unique Identifier address, this kind of address is "made" using the MAC address of the interface or device. An IPv6 address with a /64 mask requires 64 bits for the host portion and 64 bit for the network portion; the network portion is consistent across all devices in a subnet so the host portion must be unique. Since we know that MAC addresses are globally unique, we can use them to get an easy to derive MAC address. To transform a MAC address into a unique IPv6 address, go through the steps listed below; we will be using MAC address 1234:5678:90AB as an example
1. Divide the MAC address in half and insert a FFFE quartet in the middle
	   1234:5678:90AB -> 1234:56**FF:FE**78:90AB
2. Invert the 7th bit
	   1234:56FF:FE78:90AB -> 1**0**34:56FF:FE78:90AB
	   **Note:** 7th bit is in the '2' hexadecimal quartet which is 0010 and becomes 0000.
3. Append this host identifier to the subnet identifier for a complete IPv6 address.  
## Global Unicast Addresses
These are public addresses which can be used over the internet. You must register in order to use them, as they are supposed to be unique.

The address space for these addresses **used to be** 2000::/3 which gives you 125 bits of possible addresses which is a massive number, but now the range is **all addresses which have not been reserved for other purposes**.
## Unique Local Addresses
These addresses function like private IP addresses; they do not have to be globally unique, they do not have to be registered to use, but in turn, they cannot be routed over the internet and are reserved for internal use only. It is a good idea to make them globally unique anyways to account for the cases where company mergers result in subnet overlap which can cause problems.

The reserved address range is: FC00::/7 but a later update set the 8th bit to 1 so in reality, the range is FD00::/8. The network portion of the address (the first 64 bits) can be broken into 3 parts:
- First 8 bits: always set to FD signifying that this is a unique local address
- Next 40 bits: global ID; should be randomly generated to make this locally unique address into a globally unquiet address
- Next 16 bits: subnet identifier.
## Link Local Addresses
Automatically generated and assigned to IPv6 enabled interfaces. Can be enabled via the command:
	**ipv6 enable**
From interface configuration mode.

These addresses cannot be routed over the internet and are used for routing within a single subnet. The address block used is FE80::/10 however there is a technical requirement the 54 bits after FE80 be zeros which means the effective address block is FE80::/64. The host portion is generated as per EUI-64 rules. Since these packets must only be used within a single subnet, their uses are limited;
1. Routing protocols (OSPF v3 uses link local addresses for neighbor adjacency)
2. Next-hop addresses for static routes
3. Neighbor Discovery Protocol or NDP (ARP but for IPv6) uses link-local addresses.
## Multicast
Multicast addresses are used for one-to-many communication; i.e. one sender and multiple receivers but not necessarily all receivers within a domain (otherwise that would be a broadcast). The receivers are those who have joined the multicast group.

Multicast addresses use the FF00::/8 block. Here are some multicast addresses used by common protocols that are within the scope of the CCNA:
![[Pasted image 20250812115953.png]]
FF02::1 functions like a broadcast in that it sends the packet to all connected hosts.

This brings up another question; how far does a multicast travel? The answer depends on the address scope. Here are some IPv6 multicast address scopes:
- FF01: Interface local - packet does not leave the local device, can be used for inter process communication.
- FF02: Link local - packet stays within the local subnet. Routers will not forward these packets between subnets.
- FF05: Site local - packets can travel between subnets typically within one physical location/building. Should not have to be routed through WAN.
- FF08: Organization local - can include multiple sites, usually used to cover one entire AS.
- FF0E: Global - Can be routed over the internet and has no restrictions.

Multicast scopes should be defined by the network engineer and are beyond the scope of the CCNA.
## Anycast
These are also known as one-to-one-of-many or one to multiple. It differs from multicast in that it routes traffic to one router in the group, not all of them, depending on the cost/metric to each destination.

The basic operating principle is that an anycast address is a unicast address that is bound to multiple routers. A packet being forwarded out to an anycast address will go to the nearest router with that address, not to all of them.

Configuring anycast is similar to configuring a regular IPv6 address, but with the addition of the word anycast at the end; i.e.
	**ipv6 address <IP address\>/netmask anycast**
## Other Address Types
- The unspecified address (all zeros, denoted by a "::")
	- Can be used in case a device does not know its own IPv6 address
- The loopback address: ::1
	- Used to test protocols on the protocol stack.
	- only loopback address (not an entire block like in IPv4)
# IPv6 Header
![[Pasted image 20250812132520.png]]
It's different from an IPv4 header in that it has a fixed length header, and only a payload length header for the encapsulated data. It is easier for routers to process. The header fields are:
- version - 4 bits: IP version; set to 6 (0b0110) for IPv6
- Traffic class - 8 bits: used for QoS
- Flow label - 20 bits: used to identify specific traffic flows between source and destination. Which are specific lines of communication between source and destination. Similar to a TCP session I presume.
- Payload length - 16 bits: number of encapsulated data bytes (IPv6 header is not included in this field)
- Next header - 8 bits: Functions like the protocol field in the IPv4 header.
- Hop limit - 8 bits: decremented by 1 every time it is is forwarded, packet gets dropped when number gets to 0. (same as the TTL field in IPv4) 
- Source and Destination addresses.
# Neighbor Discovery Protocol (NDP)
NDP is the IPv6 replacement for ARP. It has various other functions as well. The ARP-like function of NDP uses ICMP v6 as well as ***Solicited-Node Multicast Addresses*** to learn devices' MAC addresses. NDP utilizes two messages for this purpose:
- Neighbor solicitation (NS) - ICMP v6 type 135
- Neighbor Advertisement (NA) - ICMP v6 type 136
## Solicited-Node Multicast Addresses
Thees are generated from unicast addresses, this is done by attaching the following prefix:
	**ff02::1:ff**
To the last six digits of the unicast address. So if we had a unicast address:
	*1**0**34:56FF:FE78:90AB*
The solicited node multicast address generated from it is:
	ff02::1:ff**78:90ab**
## Neighbor Solicitation Messages
They are similar to ARP requests but have some differences in what addresses are used and how they work since they rely on multicast rather than broadcasts;
![[Pasted image 20250812141023.png]]
R1 would know the IP address (unicast address) of R2 from an upper layer process (perhaps a ping request), it then uses that address to generate a solicited node multicast address (see above for how) and that IP address is the destination IP address. The destination MAC address works the same way; it is not known but a multicast MAC based on the solicited-node address of R2 is used instead.
## Neighbor Advertisement Messages
When R2 receives the solicitation message, it learns the IPv6 and MAC address of R1. It can now reply to it with a message that looks very similar to an ARP reply which contains:
- src IP: R2's IP
- dst IP: R1's IP, learned from the earlier solicitation message
- src MAC: the interface's MAC on R2
- dst MAC: R1's MAC, learned from the earlier solicitation message
## Router Discovery
IPv6 enabled routers can discover new routers that were just plugged in. To do this, they employ the following ICMP v6 messages:
- Router Solicitation - ICMP v6 type 133
	- Sent to all routers (on multicast addres FF02::2)
	- Asks routers on the local link to dinette themsleves
	- Sent when an interface is enabled/turned on
- Router Advertisements:
	- Sent to all nodes (FF02::1)
	- Sent when an interface is disabled or shutdown;
	- They are also sent as a response to router discovery message
	- The router sends these messages periodically as well
## SLAAC
Stateless Address Auto-configuration. This feature/tool allows devices to automatically configure themselves with proper subnets and addresses.

Typically, hosts use RA and RS messages to figure out what the IPv6 prefix of the local link, it then uses that address to generate a local unique address for itself using EUI-64.

What happens if two devices have the same IPv6 address? It can happen technically. This is why part of the standard is to employ **DAD** or Duplicate Address Detection. When a new interface is initialized, it sends an NS with its own IP address, if it doesn't get a reply, it means the address is unique and vice versa.
## Static Routes
Routers keep separate routing tables for IPv4 and v6. IPv4 routing is enabled by default on routers, whilst IPv6 routing must be turned on using **ipv6 unicast-routing**. Just like in IPv4 routing, in IPv6 routing, binding an interface to an IP address creates two routes, a connected route to the connected subnet, and a local route to the interface's IP address.

**Note:** Link local addresses do not appear in the routing table.
# Cisco IOS
Starting with the router, you must first enable IPv6 routing on a router (otherwise it will simply drop IPv6 packets) by running the command:
	**ipv6 unicast-routing**
in global configuration mode. Then configure the interfaces like you would regularly but instead of providing an IPv4 address and subnet, you provide an IPv6 address and subnet mask in CIDR notation, e.g.
	**int <interface\>**
	**ipv6 address 2001:db8:0:0::1/64**
	**no shutdown**

There are many IPv6 commands that are similar to IPv4 commands but using the keyword **ipv6** instead of just **ip**. You should also know that you do not need to provide a subnet mask like you would if you were configuring an IPv4 address; you can simply use CIDR notation. You can also use the abbreviated forms of IPv6 addresses, it will still work.

In order to configure an IPv6 addresses on router interfaces using EUI-64, you can issue the following command in interface configuration mode:
	**ipv6 address <host address\>/64 eui-64**

To define routes, it is similar to IPv4, the command is:
	**ipv6 route <destination/prefix length\> {next hop | exit interface \[next hop\]} \[ad]**
Depending on what is specified, it becomes a different kind of route:
- only the exit interface specified: **directly attached** route *(**These don't work in IPv6. To be clear, the command will run, and the route will be added to the routing table but the route will not work**).*
- only the next hop address specified: **recursive** route (recursive because the router has to consult the NDP table multiple times, once for the destination address, another time for the next hop address)
- Both exit interface and next hop address are specified: **Fully specified route**

In order to define a floating static route, you need to define the administrative distance of the routes to be slightly higher than the AD of the chosen dynamic routing algorithm. 