**Note:** These topics are not covered in the exam, but we are learning them anyway because Jeremy's IT Lab is awesome like that. Also because you might still get some questions about RIP and EIGRP in the exam. In depth knowledge of OSPF is required for the exam, but general knowledge about the other routing protocol is useful to know.

RIP stands for the Routing Information Protocol and EIGRP stands for the Enhanced Interior Gateway Routing Protocol.
# RIP
- distance vector interior gateway protocol. 
- RIP employs a hop-count metric where the bandwidth of the link is irrelevant and all hops are equal.
- The maximum hop count is 15, anything beyond that is considered unreachable, which limits its usage to smaller networks.
	- Rarely sees use anymore
	- Still used in home lab environments because it is quick and easy to setup.
- Three versions:
	- RIP v1 and RIP v2, used for IPv4
	- RIP Next Generation or RIPng used for IPv6
- Uses two message types:
	- Request: Ask RIP-enabled neighbors to send their routing tables
	- Response: Send the local router's routing table to RIP-enabled neighbors
- By default, RIP-enabled routers will share their routing tables every 30 seconds
	- sometimes leads to network congestion.

Let us now compare RIP v1 and v2. Starting with v1:

RIP v1:
- only advertises classful addresses (class A, class B, class C)
- does not support CIDR or VLSM (Variable Length Subnet Masks)
	- Here are some examples of the consequence of this:
		- 10.1.1.0/24 is assumed to be 10.0.0.0/8 since 10 is within the class A address range.
		- 172.16.192.0/18 is assumed to be 172.16.0.0
		- 192.168.1.4/30 is assumed to be 192.168.1.0/24
- Messages are broadcast to 255.255.255.255

RIP v2:
- supports VLSM and CIDR
- includes subnet mask information in the advertisements
- messages are *multicast* **\*** to 224.0.0.9

**\*Note:** broadcasts are sent to all devices on the local network, multicast messages are delivered only to the devices that have joined the specific multicast group. There is no need to understand how multicast groups or messages work for the CCNA.

13

To configure RIP on a Cisco router, in global configuration mode, issue the command:
	**router rip**
Within RIP configuration mode issue the pair of commands:
	**version 2**
	**no auto-summary**
The first command tells the router to use version 2 of RIP instead of the default version 1. RIP is generally not used, but RIP version 1 is never ever used. The second command instructs the router to not use classful advertisements of its interfaces, meaning: if an interface is connected to 172.16.4.0/24, the router should advertise the /24 mask and not assume it's a class B, /16 mask.

Then to activate RIP on specific interfaces, you need to issue the **network** command. To understand how it works, first look at the following diagram:
![[Pasted image 20250726124902.png]]
assuming we are configuring R1, if we issue the command:
	**network 10.0.0.0**
The router will look for any interfaces that are bound to IP addresses within 10.0.0.0/8. It will then find two interfaces; G 0/0 and G 01/0 that are bound to IP addresses within that range and activate RIP on those two interfaces, allowing them to send and receive advertisements, forming adjacency with other RIP enabled neighbors.  

**Note:** the network command is *classful* so even if you execute **network 10.0.12.0** the router will still assume its a class A network and search the entire network. This usually isn't a problem because access layer LANs use different subnets than the network layer.

**Note 2:** even though the network command is classful, since we issued the **no auto-summary** command, the advertised network will be the one on the interface, not the class A, B, or C network used in the command.

**Note 3:** the network command functions the same way in OSPF

After enabling RIP on the two interfaces connected to other routers, we need to also ensure that RIP is enabled on G 2/0. Not because it is connected to other routers, but because we want 172.16.1.0/28 to be included in the routing table and advertised to adjacent routers. We use the same network command:
	**network 172.16.0.0**

After RIP is enabled on the interface, network 172.16.1.0/28 will be advertised as well. However, we don't need to send or receive advertisements from that interface so we set it as a **passive interface**. To do that, within RIP configuration mode, issue the command:
	**passive-interface <interface\>**

**Note:** OSPF has the same feature and the same command is used as well.

A useful command to display the current routing protocol that is in effect is:
	**show ip protocols**
You can also configure some other random things about the routing protocol like maximum number of paths to the same destination you can have in the routing table, you can do so by:
	**maximum-paths <1-32\>**
This allows you to change it from the default 4 if you want more or less load-balancing.

You can also adjust its administrative distance by issuing the command:
	**distance <1-255\>**

This will allow you to make RIP routes more or less trustworthy.

To tell a router to advertise its default route, issue the command:
	**default-information originate**
from RIP configuration mode.
# EIGRP
It is Cisco proprietary for the most part, some of it was released as open-source but not all of it, so no one bothered.

It is considered a more advanced distance vector routing protocol than RIP;
- it is faster in reacting to changes in the network
- does not have the 15 hop limit that RIP has
- utilizes multi-casts sent to 224.0.0.10
- Uses ECMP with 4 paths-maximum by default like RIP but can also be configured to do unequal-cost load balancing.

To activate EIGRP on an interface, you are following similar steps; to enter EIGRP configuration mode, enter the command:
	**router eigrp <AS number\>**
The autonomous system number has to match on all connected routers for them to form an adjacency. Then, in EIGRP configuration mode, issue the **no auto-summary** command (no need for **version 2** since we are not using RIP).

You can use the network command in the same way as you did in RIP, but you can also specify a *wildcard mask* which is basically a bit inversion of a normal subnet mask. So for example, to activate EIGRP on the G 2/0 interface of R1 in the above diagram, one can issue:
	**network 172.16.1.0 0.0.0.15**

Moreover wildcard masks; the network above has a /28 subnet mask which, in decimal notation is 255.255.255.240. Bit-wise, it is:
	11111111.11111111.11111111.11110000
The wildcard mask is the inversion which is:
	00000000.00000000.00000000.00001111
Which in decimal is:
	0.0.0.15

It is best to issue the command using the network's mask but you can also just list the specific IP address bound on the interface with a wildcard mask of all zeros.

EIGRP uses bandwidth and delay to calculate its metric. In simpler terms, the metric is the bandwidth added to the delay. It is not the bandwidth of all links, just the slowest one in the route, and its also the delay of all links.

EIGRP has two distances it uses to identify the best route and the second best possible route. These two distances are:
- Feasible distance: metric of the router to the destination
- Reported distance: advertised metric of the neighboring router to the destination

The *Successor Route* is the best possible route, the one with the lowest metric, a feasible successor is one whose reported distance is lower than the successor's feasible distance. The reason for this is related to loop prevention, as it is factual that if the reported distance is lower than successor's feasible distance, the path will not lead to a route to source.

To load balance over multiple connections, we must first loosen the restriction for successor routes. To do that, we must increase the EIGRP maximum metric variance which is set to 1 by default, this effectively is ECMP (Equal Cost Multiple Paths). To allow for load balancing, we have to increase this variance multiplier. Increasing it to 2 allows paths with feasible distances less than **double** the successor's feasible distance to be included as possible paths. To increase the variance multiplier, issue the command:
	**variance <1-128\>**
in EIGRP configuration mode. 

If set to N, a feasible successor route with is one with feasible distance less than N\* Successor's feasible distance. **Note:** a route will only be used to load balance provided it is a feasible successor route. If not, it does not matter how much you toy with the variance multiplier. Reminder: for a route to be feasible, that route's reported distance must be lower than the successor's feasible distance.


