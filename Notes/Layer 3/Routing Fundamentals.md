This is an important lesson for the CCNA (two parter video on JITL). It handles the topic of routing. Routing is the process that routers use to determine the path an IP packet should take in order to get to its destination when traversing the network.

Similarly to how switches have MAC address tables, routers have routing tables. A routing table stores all the routes to all the destination known by the router. When a router receives a packet, it looks through its internal routing table.

There are two main methods for routing:
- Dynamic routing: automatically share information with neighboring routers in order to construct the routing table. An example of a dynamic routing protocol is Open Shortest Path First or OSPF.
- Static routing: manually configure the routing tables.
- A route tell the router where to send a packet with a particular destination;
	- to the next-hop. The next-hop itself is the next router along the path.
	- the destination if it is directly connected to the router
	- the router itself if the packet is meant for it

This example will be used throughout the lesson:
![[Pasted image 20250701183704.png]]

Focusing on R1, its interfaces should first be configured such that it has an IP address that corresponds to each network it is on. Once you do that (refer to [[IPv4 Addressing]] for the method), some routes will automatically become defined, even though they are automatically assigned, they aren't the product of a dynamic routing protocol, the routes in question pertain to the interfaces and their networks and are of two types:
- L - local: a route to the interface's IP address directly; netmask of 32.
- C - connected: a route to the network the interface is connected to; netmask is the network's netmask.

To use the above network diagram as an example, when configuring the IP addresses of router R1 (3 addresses total), 6 entries are added to the routing table of R1. These are 3 local addresses corresponding to the router interfaces' IP addresses (192.168.1.1/32, 192.168.12.1/32, 192.168.13.1/32) and are responsible for routing packets meant for the router itself. The /32 netmask tells the router that incident packets should have the exact same destination IP address, and it knows to receive the packet for itself instead of forwarding it through an interface. Furthermore, 3 connected addresses are defined, each corresponding to the connected network 192.168.1.0/24 for the connected LAN with PC1, 192.168.12.0/24 for the R1-R2 connection, and 192.168.13.0/24 for the R1-R3 connection. This tells the router to forward any packets meant for those subnets out of the corresponding interface.

If router R1 receives a packet meant for 192.168.1.1 What network would it match it to?
- 192.168.1.0/24 OR
- 192.168.1.1/32
The packet itself matches both routes but it can't go to both places, so the router picks the 192.168.1.1/32 because it has **more netmask bits** making it more the more specific and preferred route.

Now with that intro out of the way, we can get into:
# Static Routing
![[Pasted image 20250701183704.png]]
Using the same network, assuming we configure the IP addresses of the interfaces on all routers, each router will be able to direct packets meant either for itself or directly connected networks, however, router R1 has no knowledge of networks:
- 192.168.34.0/24
- 192.168.24.0/24
- 192.168.4.0/24
Since they are **not** directly connected. Similarly, router R4 doesn't know 3 subnets either:
- 192.168.12.0/24
- 192.168.13.0/24
- 192.168.1.0/24
and so on.

So if R1 receives a packet meant for PC4, it will not be able to route it and will instead drop the packet, even though there is a route that the packet can take to get there. Routers will not flood packets on their interfaces.

If end hosts want to communicate within their LAN, they can rely on IP addresses and ARP requests to get the information they need to communicate, but if an address is not within the network, but is nonetheless reachable, the packet will definitely pass through the default gateway. End hosts, like PC1 and PC4 have a *default gateway* which is a route that is used if a packet is sent outside the network. Within the routing tables of PC1 and PC4 would be the following entries: 
![[Pasted image 20250702140629.png]] ![[Pasted image 20250702140640.png]]
This is a linux PC by the looks of it, as indicated by the eth0 interface name which is a common name used by linux distros to name their wired interfaces (similar to how Cisco products default to using G0/0, G0/1, etc...). Breaking down the lines of the image,
- *iface eth0 inet static* - interface named eth0 was statically configured
- *address 192.168.1.10/24* - IP address of host is 192.168.1.10 and other hosts within the same LAN are in the 192.1268.1.0/24 network.
- *gateway 192.168.1.1* - If a packet is meant for a destination not within 192.168.1.0/24 send it to this IP address (which is the router and is responsible for routing the packet externally)

Now if PC1 tries to communicate with PC4, it will send the packet(s) to R1 which, as was stated earlier, does not know PC4's network or a route to it which means it will drop the packet. To allow for proper forwarding, a route has to be configured on R1 to forward a packet to the next hop. For the sake of simplicity, the route that will be taken from PC1 to PC4 and vice versa is through R3, i.e. PC1 -> R1 -> R3 -> R4 -> PC4 and PC4 -> R4 -> R3 -> R1 -> PC1. In reality, the other path through R2 can be used as well either as:
- load balancer: if R1 -> R3 becomes too congested, traffic can be routed through R2.
- Backup: if R3 goes down, R2 can replace it.

To allow for two way reachability between PC1 and PC4, the following routes must be configured statically into Routers R1, R3, and R4 (R2 is left along for now):
![[Pasted image 20250702142040.png]]

## Cisco IOS Commands
In privileged exec mode, you can run:
	**show ip route**
To show the router's routing table. To add a route, for example, if we want to add the route from R1 to 192.168.4.0/24 to R1's routing table, in global configuration mode, you run:
	**ip route <destination ip\> <netmask\> <next-hop's IP\>**
For the above example, this command will be:
	**ip route 192.168.4.0 255.255.255.0 192.168.13.3**
This will cause another route to appear in the routing table (highlighted in cyan)
![[Pasted image 20250702142856.png]]
The 'S' indicates that this is a static route. The \[1/0\] after the IP address stands for: \[Administrate distance/Metric]. More on these concepts later.

 Instead of specifying the IP address of the next hop, you can instead specify the exit interface which is the one that packets will exit through, on their way to the next hop. The command is:
	 **ip route \<destination ip> \<netmask> \<exit inteface>**
Using R2 as an example, it can be connected to PC1's LAN via the command:
	**ip route 192.168.1.0 255.255.255.0 g0/0**

You can even specify both the exit interface and the next-hop IP address in the command:
	 **ip route \<destination ip> \<netmask> \<exit inteface> \<next hop IP address>**
R2 can be connected to PC4's LAN using:
	 **ip route 192.168.4.0 255.255.255.0 g0/1 192.168.24.4**

These commands will all work and add routes function identically. When displaying the routing table, the differences are minor:
![[Pasted image 20250702144813.png]]

The address highlight in green is the one added in the last example and only differs from the one above it in that it has both the interface ID as well as an IP address. Furthermore, if the interface alone is defined, the routing table will display:
	\<network identifier> is ***directly connected***
Even though it isn't in the instance above. Adding the full line (exit interface and IP address) will yield the correct information which is that a subnet is not directly connected, but instead reachable via another node.

When you define the exit-interface only, routers rely on a process called [proxy ARP](https://www.cisco.com/c/en/us/support/docs/ip/dynamic-address-allocation-resolution/13718-5.html). It is thus good practice to define the next hop's IP address, or both the exit interface and IP.

Default gateways or default routes, in technical terms, are routes to 0.0.0.0/0 or put more simply, a route to everywhere that is not at all specific. Default routes are configured so that packets that cannot be routed anywhere can go there. Typically, in home networks and plug-and-play routers, the default gateway of your home computer will be that of the router, and the router's default route or default gateway leads to the internet. In an enterprise setting with multiple separate LANs, the router checks to see if a packet can be routed to any of the internal networks, if not, it is routed to the internet.

Configuring a default gateway can be done explicitly via:
	**ip route 0.0.0.0 0.0.0.0 <IP address of next-hop\>**
The IP address needed is usually given to you by your ISP (in the case where default-gateway means internet). On the routing table, this route will show up with a '**S\***' symbol; the 'S' meaning static, and the '\*' meaning it is the "candidate default". You can have multiple candidates for default routes.