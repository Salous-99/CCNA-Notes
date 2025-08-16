Open Shortest Path First or OSPF is the algorithm that is included in the CCNA, it is also a quick and reliable method of dynamically filling routing tables. It is an example of a Link State IGP, and it is an industry standard. 

OSPF utilizes Dijkstra's algorithm as a means of constructing the network map with the associated metric. OSPF has three versions, the v1 is not used anymore because its old, v2 is traditionally used for IPv4 networks and v3 is used for IPv6 networks but can be used for IPv4 as well (although v2 is more typically used for IPv4). OSPF v2 is the one we will discuss here in more depth.

OSPF enabled routers store information about the network in the form of an LSA or *Link State Advertisement*. These LSAs are stored in an LSDB which is a *Link State Database*. routers will flood LSAs until all routers in the *OSPF area* develop the same LSDB.

Consider the following network that we will use for demonstration purposes:
![[Pasted image 20250729152409.png]]
Let us assume that at this point, all routers have developed their own LSDB and have populated their routing tables and are in a stable state. It is at this point that R4 is connected to a LAN with network address 192.168.4.0/24 like so:
![[Pasted image 20250729152516.png]]

When this happen, a new LSA is created containing the following:
- R4 router ID
- Network address for the newly connected LAN (192.168.4.0/24)
- Cost of R4 to the LAN

This LSA will then be flooded to all routers and is added to their respective LSDBs if they are within the same OSPF area.

Only single area OSPF is required for the CCNA, but the idea behind distinct OSPF areas is to manage very large networks more smoothly. If a large network (500+ routers) were to use single area:
- SPF algorithm takes more time to calculate routes
- SPF algorithm requires exponentially more processing power per additional router
- The LSDB will take up more memory on each router
- Any change in the network will cause LSAs to be flooded to many many routers, possibly causing congestion.

An OSPF area is defined as one were all the routers share the same LSDB. Typically, OSPF networks divide the network into multiple OSPF areas in accordance with the following:
- OSPF area 0 is the **backbone area** and it is an area that all other routers must connect to through at least one ABR (see below).
- Routers with all interfaces within the same area are called **internal routers**
- Routers with interfaces in different areas are called **Area Border Routers** or ABRs.
- Routers connected to the backbone area are called **backbone routers** (even if only one interface is connected, it would still be a backbone router)
- an inter-area route is a route to a destination inside the same OSPF area
- an intra-area route is a route to a destination inside a different OSPF area
- All OSPF areas must be contiguous (routers cannot be physically or logically disconnected from one another)
- OSPF interfaces in the same subnet must also be in the same area.
## OSPF Neighbors
Routers become neighbors with one another when they exchange OSPF hello messages. These are sent out of OSPF enabled interfaces every 10 seconds (determined by the Hello timer. Set to 10 seconds by default). OSPF hello messages are multicast to the address 244.0.0.5.

These OSPF messages are encapsulated with an IP header with a protocol field of 89 for the OSPF protocol.

Assuming we have two routers connected to one another on OSPF enabled interfaces; if that is the case, whenever one of the routers reaches the hello timer it sends a hello message on the interface to its neighbor before knowing whether or not it has a neighbor. Because neither router knows about the other, the first hello message sent will contain the sender router's ID and a neighbor router ID of 0.0.0.0. When the neighbor receives this message, it will add the sender router's ID to its LSDB and respond with a hello message with its own ID as well as the correct neighbor ID (it now knows this ID because it got the hello message). Finally the first router will resend a hello message with the correct IDs. So to recap:
- Down state: neither router knows about the other
- Init state: first hello message is sent, one router knows about its neighbor but the other router doesn't
- 2-way state: both routers know of their neighbors existence as well as their router IDs.

After that, the routers are ready to exchange their LSDB information. Before doing that, the routers must establish which router is the slave and which is the master. They do so in the *Exstart* slave. The router with the higher ID is chosen as the master and the other, the slave. To do this comparison, the two routers exchange Database Description messages or DBD messages.

The routers then enter the *Exchange* state in which they exchange LSA information. **Note:** the LSA information shared is not the entire LSDB, but instead, the two routers tell one another which LSAs they have so that the other router can respond with what LSAs they need. The actual LSDB exchange happens in the next state which is the **loading** state. In this state, both router exchange Link State Requests or LSRs to each other so that they can fill out their LSDBs with entries that are not there yet. That way, both routers will have the same LSDB. The response message to an LSR is called a Link State Update message or LSU. Upon receiving an LSA, a router will respond with an LSAck or Link State Acknowledgment.

The final state is the *Full* state in which routers have the full LSDB within memory. Hello messages are still exchanged even in the full state to ensure that links are intact. If no Hello packets were received on a link for a specific amount of time (indicated by the *Dead* timer and is set to 40 seconds by default), that means an adjacency was lost. The timer is reset whenever a Hello message is received (which by default happens every 10 seconds). Note that the 40 and 10 second timers are specific to OSPF running on an Ethernet interface. These timers values can also be set at 120 and 30 for Dead and Hello respectively over other interfaces.
## OSPF Network Types
There are three main OSPF network types:
- Broadcast: enabled by default in Ethernet and Fiber Distributed Data Interfaces (FDDI)
- Point-to-point: enabled on Point-to-Point (PPP) and High-Level Data Link Control (HDLC) interfaces
- Non-broadcast: enabled by default on *Frame Relay* and X.25 interface (these types are the ones that use 120 and 30 second timers)
### Broadcast
CCNA covers the broadcast and PPP network types; starting with the broadcast type, consider the following OSPF network:
![[Pasted image 20250803121537.png]]
The labels DR, BDR, and DROther stand for Designated router, backup designated router and designated router other respectively. The rule is: every subnet must have one DR, one BDR, and the rest of the connected routers on the subnet should be DROther. The router election happen in accordance with the following priorities:
- Highest OSPF priority router is designated as the DR, and the second highest as the BDR.
- Since all OSPF priorities are the same by default (set to 1), you can go by highest router ID instead.*
- All other router interfaces are then set to DROther.

**Note:** You can change the OSPF priority of an interface by running the command:
	**ip ospf priority <0-255\>**
in its interface configuration mode. Setting the priority to 0 means that this router cannot be the DR or the BDR, no matter what.

Changing the OSPF priority of a router in a live network will not change its role as DR/BDR election is non-preemptive. Once the DR/BDR are elected, they will not change until a reset occurs or a link fails. So if The priority of R2 were to be increased beyond 1, it will not immediately be designated as the DR. In fact, nothing will happen until a manual reset is performed that triggers reelections. After that, we will end up in this situation :
![[Pasted image 20250803124040.png]]
Two things to note and understand:
- R4 became the DR, not R2 even though R2 has the highest priority
	- After the reset R4, which used to be the BDR, immediately became the DR (that's how backups work)
	- The reelection that was held elected the highest priority router as the **Backup** DR.
-  The neighbor state between R3 and R5 is in the two state.
	- DROther routers will move to the full state only with DR and BDR routers, it will be in the 2-way state for other DROther routers. 
	- This is done so that the network is not flooded with two many LSAs.

All routers will still obtain the same copy of the LSDB. However, by limiting communication to only the DR and BDR, some unnecessary network traffic is cut out.
### Point-to-Point
![[Pasted image 20250803125800.png]]
The above network has one Ethernet connection replaced with a serial connection. A PPP network type is enabled on serial interfaces using the PPP or HDLC encapsulation protocols. In a PPP network, no DR and BDR are elected since there is no need to; the routers will form a full adjacency with one another regardless.

Serial connections are deprecated and have been removed from the CCNA exam topics. Here is a brief overview:
- The default encapsulation used on serial connections is HDLC (on Cisco routers, it is cHDLC which stands for Cisco HDLC)
	- To change to PPP encapsulation, you can run the command **encapsulation ppp** on the router interface configuration mode.
- A serial connections has two sides; the Data Communications Equipment or DCE and the Data Terminal Equipment or DTE:
	- DCE is responsible for picking the clock rate (bandwidth) and communicating that to the DTE
	- The clock rate is set using the command **clock rate <clock rate\>**
	- You can identify which interface is which by running the command **show controllers <serial interface\>**

**Note:** You can manually configure a Point-to-Point connection to be a PPP network type if you issue command:
	**ip ospf network <type\>**
On the router interfaces of the point to point connection. This, however, is not necessary because if you set the network type to PPP or broadcast, a full state will be reached anyway. 
## OSPF Metric
How does the OSPF calculate its needed metric? It does so by dividing the bandwidth of the interface with a reference bandwidth that is set to 100 Mbps by default. Everything smaller than 1 is rounded up to 1 so the cost of a Fast Ethernet (100 Mbps link) is 1, but Gigabit Ethernet is also 1 even though 100/1000 = 0.1, the cost is rounded up to a minimum of 1. If the reference bandwidth is too large and causing all links to be treated equally, you may change it using the command:
	**auto-cost reference bandwidth <1-4294967\>**
Where the number is the reference bandwidth measured in Mbps.

By setting the reference bandwidth to something larger than 100 Mbps, perhaps 1000 Mbps, then there will be differentiation between Fast Ethernet and Gigabit Ethernet as they will have costs 10 and 1 respectively. A rule of thumb is to set the reference bandwidth to a value greater than the bandwidth of the fastest link in your network. This ensures that no link-type costs are equal (Fast Ethernet is distinct from Gigabit Ethernet is distinct from 10 Gigabit Ethernet, etc...), it also ensures that your network has room for upgrades (faster links). When changing the reference bandwidth, it is important to change it on all routers within the same area, otherwise the advertisements any one routers makes will not be reliable.

You can also manually set the cost of a link by going to interface configuration mode for the interface in question then issuing the command:
	**ip ospf cost <1-65535\>**
This overrides the calculated cost as per OSPF protocol. You can also change the bandwidth of the interface manually, thus affecting its OSPF cost, using the command:
	**bandwidth <1-10000000\>**
Run from interface configuration mode of the interface in question. **Note:** This will not change the interface speed (you must use the **speed** command for that) it will only change the value that is used for OSPF (and other) calculations. Since this bandwidth is used in multiple calculations, it is best to not change it. For manual configuration of OSPF cost, it is best to use **ip ospf cost N**.

What about a loopback interface with no physical link?? What about any virtual interface? What is its metric if it doesn't have a baud rate? It's 1 by default.
## OSPF Neighbor Requirement
1. Area numbers must match
2. Interfaces must bu within the same subnet
3. OSPF process must not be shutdown
4. OSPF router IDs must be unique
5. Hello and Dead timers must match
6. Authentication setting must match*
7. IP MTU settings must match
8. OSPF network type must match

Requirements 7 and 8 can be ignored and routers will still find and form an adjacency with their neighbors but OSPF will not work properly.

**\*Note:** To set an OSPF password and enable authentication, run the commands:
	**ip ospf authentication-key <password\>**
	**ip ospf authentication**
From interface configuration mode. The first command sets the password and the second command enables authentication on that OSPF interface.
## LSA Types
There are 11 LSA types, only 3 of which are relevant for the CCNA:
1. Type 1: Router LSA - generated by all OSPF enabled routers. Identifies the router using its ID and advertises all networks attached to OSPF enabled interfaces on the router.
2. Type 2: Network LSA - Generated by the DR of each multi-access (broadcast type) network. Lists the routers attached to the multi-access network.
3. Type 5: AS External LSA - Generated by the ASBR and describes routes to destinations outside of the AS.
## Cisco IOS Configuration
To configure OSPF on the routers interface, enter global configuration mode and issue the command:
	**router ospf <process ID\>**
The process ID is locally unique as you can have multiple OSPF processes running simultaneously on a single router. They are not unique within the network topology though (like an AS is for EIGRP) and OSPF routers with different process IDs can still from an adjacency with one another.

Then enable OSPF on the router's interfaces by entering the command:
	**network <interface address\> 0.0.0.0 area <area number\>**
You can also directly enable OSPF on an interface from its interface configuration mode using:
	**ip ospf <process-id\> area <area No.\>**

Two things to note real quick:
- OSPF also uses wildcard masks which means you can enter the subnet network address and wildcard mask instead of doing what is above (easier to do it this way if you know the IP address configured on the interface)
- The area number must be supplied here so that the interface can also be bound to a specific area.

The **passive-interface** command works the same as in EIGRP or RIP. i.e.
	**passive-interface <interface\>**
From OSPF configuration mode, or by running the command:
	**passive-interface default**
Which sets all interfaces to passive, then you can individually run:
	**no passive-interface <interface\>**
On the interfaces that should send OSPF messages. I do not know why you would do it this way.

When a default route is configured onto a router, you can advertise that default route to connected routers via the same command: **default-information originate**. This will cause the router to create a new LSA and flood it. Note that a router with a default route configured on it is called an *Autonomous System Boundary Router* or ASBR.

Also note that OSPF has similar system for determining router ID, if no ID was manually configured, the router ID will be:
- Highest IP address on a loopback interface
- Highest IP address on a physical interface

To manually configure the ID, enter the command:
	**router-id <Router ID\>**
In OSPF configuration mode. The ID must have the same format as an IP address. **Note:** This differs *slightly* from EIGRP where the command there is:
	**eigrp router-id <Router ID\>**

When using OSPF, after setting the router ID, we must reset the OSPF process so that the change can take effect. This can be done via restarting the entire router (turn on and off) or by the command:
	**clear ip ospf process**
From privileged exec mode. This will restart the process as well as **dump all LSAs within the LSDB** causing the router to be bad at routing for a short period of time until the LSDB is repopulated and stable.

OSPF does not support unequal cost load-balancing. It supports ECMP over 4 paths by default, to change the maximum number of paths, issue the command:
	**maximum-paths <1-32\>**
from OSPF configuration mode. Changing the administrative distance is also identical to RIP:
	**distance <1-255\>**
from OSPF configuration mode.

To display information about the routing protocol, you use the command:
	**show ip protocols** (same command used regardless of what protocol is enabled)

To display the OSPF database specifically:
	**show ip ospf database**
This will display all LSAs on the router.

You can run:
	**show ip ospf neighbor**
To display immediate neighbors to this router that are OSPF enabled. There is also:
	**show ip ospf interface <interface (optional)\>**
That displays the OSPF setting on the specified interface or all interfaces if no interface was specified in the optional field. Or:
	**show ip ospf interface brief**
for a quick overview.

