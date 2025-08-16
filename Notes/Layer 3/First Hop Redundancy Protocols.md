![[Pasted image 20250804121338.png]]
Let's say that the default gateway is R1, and all the end hosts are configured to send traffic to 172.16.0.254 whenever traffic is sent to the internet. What happens if R1's interface goes down? Or if the wire is damaged? Traffic should be directed through R2 which serves as a backup, the only issue is that the end hosts have a statically configured default gateway of 172.16.0.254.

An FHRP is a protocol designed to protect the default gateway. In the event that a router failure, a backup router will automatically take over the address within a few seconds. The default gateway is considered the "first hop" for traffic to get to wherever its going, hence **first hop** redundancy protocol.

How can this work? Different FHRPs behave differently, but in general, it works as follows;
- Both routers create the same virtual interface bound to a VIP (Virtual IP address) and a virtual MAC address.
- The routers also agree on which one is active and which one is waiting in standby mode.
- The routers continuously exchange hello messages for the purposes of ensuring that the other is still active (will become relevant later)
- This VIP is configured as the default gateway on all end hosts.
- When an end host wants to send traffic it sends an ARP request for the VIP.
- The ARP request is flooded on the network, reaching both routers
- Only the active router will respond to the request with the virtual MAC address
- If the active router goes down, the standby router will go live
- The now-active router will send *gratuitous ARP replies* to all end hosts, this ARP reply will also have the virtual MAC address
	- all switches receiving a frame from the same MAC address but on a different interface will now change their MAC address tables to reflect the new route
	- end hosts connecting to the internet still send traffic to the same MAC address (the virtual MAC address) but because of the gratuitous ARP reply, the switches within the network will forward the frame to a different destination.

**Note:** Even if the previously active router goes back up again, it will not preempt the current active router unless it was configured to do so.
## HSRP
Hot Standby Router Protocol is a Cisco proprietary protocol with two versions, the first for IPv4 and the second with IPv6 support.

Things I need to memorize for some reason:
- "Active" and "Standby" are the terms used to describe the routers' states
- More groups that can be configured on HSRP v2.
- Groups allow load balancing by electing different active routers in each group
- Multicast addresses:
	- v1:
		- IPv4 multicast address (used for reeving "Hello" messages): 224.0.0.2
		- Virtual MAC address 0000:0C07:AC**XX** Where the last two characters are the group number (two hexadecimal characters implies a maximum of 256 possible groups)
	- v2:
		- IPv4 multicast address: 224.0.0.102
		- Virtual MAC address 0000:0C9F:F**XXX** (4096 possible groups)
## VRRP
Virtual Router Redundancy Protocol is not Cisco proprietary but is very similar to HSRP. Key differences:
- Master and backup instead of active and standby
- 224.0.0.18 for multicast hello messages
- 0000:5E00:01**XX** for the virtual MAC address
## GLBP
Gateway Load Balancing Protocol is also Cisco proprietary, but differs from HSRP in that it allows for multiple routers to load balance within the same subnet. Works similarly to HSRP with some key differences:
- single subnet load balancing 
- designates an Active Virtual Gateway or AVG
- The elected AVG designates up to 4 Active Virtual Forwarders or AVFs (it can designate itself also)
- Each AVF acts as a default gateway for a portion of the end hosts in the subnet.
- IPv4 Multicast address: 224.0.0.102
- Virtual MAC address: 0007:B400:**XXYY**
	- XX: GLBP group number
	- YY: AVF number

## Configuring HSRP on Cisco IOS
This is not needed for the CCNA, but here is some basic configuration;

Version 1 is enabled by default, to enable version 2, run the command:
	**standby version 2**
From interface configuration mode (Yes, on an interface basis). Also, versions must be the same across different routers for HSRP to work. 

To actually enable HSRP, first set the VIP by:
	**standby <group number\> <VIP\>**
**Note:** The group number is 0-255 or v1 and 0-4095 for v2. The VIP is chosen by you.

Set the priority of the router interface (to force it to be picked as the active router):
	**standby <group number\> priority <0-255\>**
The highest priority router will be the active one. If priority is equal, the highest IP address router will be the active router.

You can enable preemption by running
	**standby <group number\> preempt**
**Note:** The group number must match between different commands for any one group.

You can display HSRP status by running:
	**show standby**
