What we have previously seen in [[Routing Fundamentals]] are examples of static routes that are manually configured on the router(s) to direct traffic according to a predetermined path. You can also configure dynamic routing protocols on routers that automatically populate their routing table and respond to any changes in the network topology.

Dynamic routing protocols work via sending advertisements to one another. Some interfaces, at least 2, will be configured on the router's interface, thus populating its routing table with 4 routes automatically; these are the 2 directly connected routes to the networks on each interface, as well as the 2 host addresses that were defined on each network interface. These routes are then advertised to neighboring routers and are added to their routing tables, and these same advertisements are then forwarded to other routers, allowing a network of connected routers to learn the best routes to reach any one LAN.

If network failures were to occur, this information would also be propagated throughout the rest of the -intact- network so that routers can route traffic through the second next path. It is important to build redundancy in your network for the purposes of ensuring that there is a second best path always available. Consider the following network: 
![[Pasted image 20250723121329.png]]
If a failure were to occur on the connection between R4 and R2, subnet 192.168.4.0/24 will become unreachable. It is better to connect the routers like so:
![[Pasted image 20250723121551.png]]
In case of failure, traffic from R1 can still be routed through R3.

There are two kinds (for all I know) of routing protocols that are within the scope of the CCNA exam, these are:
- Interior Gateway Protocol (IGP)
- Exterior Gateway Protocol (EGP)

The former dynamic routing protocol is used to determine routes within a single *Autonomous System* or AS, such as a single organization or company. Whilst EGP is used to determine the best routes between different autonomous systems.

rlly good slide:
![[Pasted image 20250723122318.png]]
Some things to note:
- Details of how a Path Vector algorithm works are not needed for the CCNA, neither is the Border Gateway Protocol. You simply need to know that there is one algorithm for EGP which is the Path Vector algorithm and that the Border Gateway Protocol is the standard EGP.
- Intermediate System to Intermediate System (IS-IS) is not covered in the CCNA, but is covered within the [CCNP Service Provider](https://www.cisco.com/site/us/en/learn/training-certifications/certifications/service-provider/ccnp-service-provider/index.html) Path.
## Distance Vector Algorithms
They are named so because each router only knows the cost of taking a specific route (Distance), as well as the direction (Vector). Routers using distance vector algorithms such as RIP and Cisco's proprietary IGRP only advertise their available routes to their immediate neighbors and only hear from those neighbors to build their own routing tables.

The routers send the following information to their directly connected neighbors:
- Their known destination networks
- Their metric to reach their known destination networks
## Link State Algorithms
Routers using these algorithms will send advertisements to their connected neighbors and their neighbors will then add their own cost onto it; for example if R1 can reach subnets A and B with costs 4 and 5 respectively and it advertises this information to R2, the link cost between R1 and R2 is 2 so that means the cost is 6 and 7 for reaching subnets A and B respectively through R2. This amended advertisement is then forwarded to the connected neighbors of R2. Overtime, all the routers will build a complete picture of the network and will individually make routing decisions. This takes up more CPU power and resources but has the benefit of reacting faster than distance vector algorithms when a change occurs in the network.
## Metrics and Administrative Distance
Different algorithms employ different methods of calculating the metric which is the cost of sending traffic through a specific interface.

(Unsure if I can make a general statement here) Routers will add to the shortest path to their routing table, and if two routes have the same cost, then both are added and traffic is load balanced between them, provided they have the exact same destination (same network address and mask). This method of load balancing is called: Equal Cost Multi-Path or ECMP.

Here are how the different IGPs calculate cost:

| IGP   | Metric            | Explanation                                                                                                                           |
| ----- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| RIP   | Hop Count         | Each router in the path counts as one hop. The total number of hops is the cost. All links are considered equal.                      |
| EIGRP | Bandwidth & Delay | A formula that takes as inputs, the delay of the slowest link in the route as well as the total delay of all links, and other things. |
| OSPF  | Cost              | Sum total cost of each link in the route                                                                                              |
| IS-IS | Cost              | Sum total cost of each link in the route, but links have a cost of 10 by default.                                                     |
In most instances, a single AS would use one routing protocol, but in some instances a company might use two, perhaps due to their large size, or because they are sharing information with another AS, or any other reason. In these cases, an AS might use two algorithms; if that is the case, how does the router compare cost? It uses something called *Administrative Distance* this is shown next to the cost (number on the left) when displaying the routing table on a Cisco router. A lower administrative distance means a more trustworthy and preferred route. Here is a table of some AD values for common routing protocols:
![[Pasted image 20250723132115.png]]
Notice that static routes have an AD of 1, that is the second most trustworthy after directly connected networks. You might want to ensure that the algorithm is taking care of the routing and that static routes are used as a backup, so you may want to increase the AD of the static route. Firstly, you can do that simply by adding the AD you want after the IP route command like so:
	**ip route A.B.C.D A.B.C.D A.B.C.D <AD\>**
Where the first three addresses are the destination, subnet mask, and route respectively. By setting the AD to 100 for example, and use the EIGRP protocol as the IGP, you are effectively asking the router to favor the EIGRP route (with AD = 90) over the static route. If the dynamic route goes down, you still have the static route. This is known as a **Floating Static Route***.