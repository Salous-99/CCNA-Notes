When designing networks from scratch, there are best practices that can inform the engineer's design choices, however, there are few universal correct answers and the design choices you make will depend on the network's usage and vary from case to case.

Designing networks from scratch is beyond the CCNA but understanding a few basic design concepts is relevant.
# Common Network Topologies
## Start Topology
![[Pasted image 20250907195646.png]]
Several end hosts connected to one central switch (is not always drawn in a star shape).
## Full Mesh Topology
![[Pasted image 20250907195732.png]]
Each device is connected to each other device
## Partial Mesh
![[Pasted image 20250907195755.png]]
Some devices are connected to all others, some are partially connected.
# Two-Tier LAN Architecture
The two tier design is called so owing to its two kinds of switch layers, the two being:
1. Access Layer
2. Distribution layer
This design is also called a *collapsed core* design due to it not having the third layer that is known as the *core-layer*
## Access Layer
- The layer that connects to end hosts PCs, Cameras, Printers, etc...
- The switches have a lot of access ports so that they can connect to the many end hosts.
- QoS is typically done here.
- Security services like DAI, DHCP Snooping, etc... are performed within this layer.
- Switchports might be PoE enabled for wireless access points, IP phones, etc...
## Distribution Layer
- Aggregates connections from the access layer switches.
- Typically is the border between layers 2 and 3.
- Connects to the internet/WAN.
## Two-Tier Campus LAN Design
![[Pasted image 20250907201428.png]]
This is a typical design for a two-tier system. STP might disable some ports on the access switches, but that is to be expected. The two upper distribution switches will use some [[First Hop Redundancy Protocols]] for example: Hot Standby Router Protocol or HSRP if they are Cisco Switches, to balance traffic between the two default gateways.

This network might also connect to another network that stores servers for this organization:![[Screenshot from 2025-09-07 20-15-00.png]]
The connections between the two distribution layers are most likely routed/IP-enabled ports and share routing information via OSPF or another dynamic routing protocol. The distribution layer is sometimes known as the Core-Distribution layer.

**Note:** The connections between the switches in the access and distribution layers form a partial mesh topology, and the end hosts connected to the access layer switches form four small star topologies. The topology between all the distribution layer switches is a full-mesh topology since every layer 3 switch is connected to every other layer 3 switch.

When relying on a two-tier topology, increasing the number of distribution layers will cause the number of connections needed to connect all of them together to rise rapidly, this makes it difficult to scale the network up. In fact, after 3 distinct distribution layer, Cisco recommends adding a third *core layer* that uses very fast network devices to route information between different distribution layers. It also has the benefit of saving up on connections and ports.
# Three-Tier LAN Architecture
Continuing from the above point, consider a situation where there are 6 distinct distribution layers for a given organization, if so, the connections will look like this:
![[Pasted image 20250907203314.png]]
Many many connections. This can be made a whole lot simpler if a core layer is added:
![[Pasted image 20250907203339.png]]
No need for a full-mesh connecting all the distribution layer switches.

**The Core Layer**:
- Connects distribution layers together in large LANs
- Speed focused
- CPU-intensive operations like security, QoS marking and classification, etc... should be avoided at this layer to maximize speed.
- All connections are layer 3 connections. No STP, no blocked ports. *Millions Must Forward*.
- Should maintain connectivity throughout the LAN even if a device fails.
![[Pasted image 20250907205707.png]]
This here shows how a core layer would be integrated into the setup from earlier. The two core layer switches would connect to other distribution layer switches like so: ![[Pasted image 20250907210308.png]]
# Spine-Leaf Architecture
Data centers used to use a three-tier architecture which worked well given that most traffic moving across data centers was *North-South* meaning to and from the servers and WAN, but no so much East-West or intra-LAN traffic. With the advent of virtual servers, applications are deployed across many physical servers which increased the east-west traffic in the data center which in turn increased latency and variability in the server-to-server latency depending on the path the traffic took. To solve this, the Spine-Leaf Architecture (also called Clos architecture after Charles Clos who formalized the concept) was introduced and became more prominent in data centers.

The structure looks like so:![[Pasted image 20250907211745.png]]
The idea is simple: every leaf switch is connected to every spine switch, and every spine switch is connected to every leaf switch. Leaf switches aren't connected to each other, and spine switches also aren't connected to each other. Finally, end hosts connect only to leaf switches.

This reduced variability for east-west traffic as all traffic travelling east-west passes through, at most, 3 switches (1 switch if connected to same leaf). The path taken by the traffic is randomly chosen for consistent latency and load-balancing among the spine switches.
# Small Office/Home Office Setup
Also known as SOHO, this setup is common to all small enterprises with no high bandwidth applications, for example a normal internet connected home setup. In a home or small office environment, there isn't any real need for separate switches, routers, wireless access points, DHCP and DNS servers, firewalls, etc... so instead, they are all wrapped into one appliance known commonly as a home router or wireless router. This wireless router is also plug-and-play meaning it requires almost no setup or maintenance.