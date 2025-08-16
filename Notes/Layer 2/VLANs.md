Virtual LANs allow us to segment a network on the same switch. What is a single LAN? A LAN is a single **broadcast domain**, which is a group of devices which will receive a broadcast frame sent by any one member of the group. Finally, a broadcast frame is a frame which is sent to destination MAC address FFFF:FFFF:FFFF.

Consider a situation where one office with 10-12 end hosts, split between 3 departments share one switch; if a member of one department sends a broadcast frame meant for the other member of their department, all hosts will receive it, incurring unnecessary overhead traffic and reducing overall network performance. There are also security concerns, by segmenting the network, you can apply firewall rules that mediate access between the subnets, this control is not present when within the same LAN because traffic sent between hosts on the same LAN does not go through a router or a firewall. Furthermore, limiting the access of any one user is good security practice; someone in the HR department has no reason to access files in the R&D department.

Here's the wrong way to segment a network on the same switch:
![[Pasted image 20250707133923.png]]
The idea here is configuring each host with the IP address, subnet mask, and default gateway for a separate router interface, connecting three different interfaces on the router to the switch, and having the segmentation happen virtually only. There is one issue with this setup however, the first is broadcast frames or unknown unicast frames will still be flooded out of all interfaces. The second issue is a security one because you still have access to the devices connected to the same switch and you can, after doing some reconnaissance to find out the target host's IP and MAC address, you can send it traffic directly.

You can make the above setup work by purchasing three separate switches, although for a small organization, this would be too costly. This is where VLANs come in:
![[Pasted image 20250707134536.png]]
You can configure every interface on the switch to be part of one VLAN, making the connected end host a part of that VLAN. The switch will consider each VLAN separate and will not forward traffic between them, even if the destination MAC address belongs to an end host in a different VLAN, including unknown unicast and broadcast frames.

However, all inter-VLAN traffic must still go through the router, packets are forwarded to the subnet's interface, the router forwards it again to the destination host through the switch. **Note:** even if two hosts were to be in the *same subnet* but on *different VLANs*, a switch will not forward traffic directly between them; as far as the switch is concerned, only layer 2 information is needed to make the decision on where to forward it.

Okay now moving on from this primitive tech, we can use ***trunk ports***. trunk ports carry traffic for multiple VLANs and allow for a cleaner network topology. Consider the following network:
![[Pasted image 20250707150703.png]]

You could very well have two switches separating devices on the same VLAN, this can happen as the result of the geography (same department, different floors, different office buildings, etc...). When virtually segmenting a *small* network, this setup would do, but as the number of VLANs increase, so too would the number of wasted switch and router interfaces wasted. To solve this, we can use ***trunk ports*** to direct VLAN traffic through a single interface only:
![[Pasted image 20250707151050.png]]
All traffic can be directed through one interface and identified via a piece of information added onto the Ethernet frame header as per the 802.1Q standard (previously it used to be the ISL or Inter-Switch Link, no longer supported, not even by Cisco).

 When traffic leaves through a trunk port, it is "tagged" in order for the receiving device to know which VLAN it belongs to. This tag is inserted between two fields of the Ethernet header. The header format is:
- Preamble
- SFD
- Destination
- Source
- Type/Length
The additional tag (4 bytes long) is inserted between the Source and Type fields of the Ethernet header, like so:
![[Pasted image 20250707151631.png]]
The tag itself consists of two main fields:
- 2 bytes - Tag Protocol Identifier (TPID)
- 2 bytes - Tag Control Information (TCI)

TPID is always set to 0x8100 which indicates the protocol being used is 802.1Q.

The TCI consists of three subfields:
- 3 bits - PCP/Priority Code Point: used for Class of Service (CoS), prioritizes important traffic in congested networks.
- 1 bit - DEI/Drop Eligible Indicator: used to indicate that a frame can be dropped if the network is congested.
- 12 bits - VID/VLAN ID: Identifies the VLAN the frame belongs to.
	- 12 bits allows for 4096 ID
	- ID 0 and 4095 are reserved
	- 4064 possible VLANs
		- Normal VLANs: 1 - 1005
		- Extended VLANs: 1006 - 4094
		- Some older devices cannot access the extended VLAN range.

Finally the 802.1Q standard (not present in ISL) has a concept of a *Native VLAN*; switches will not add the tag to frames in the native VLAN. Switches that receive untagged frames will assume that the frame is meant for the native VLAN as well. This means that if you have connected switches, you must ensure that their native VLANs match.

**Note:** Native VLANs are set to VLAN 1 by default on all open trunk ports but can be configured manually on each trunk port. This is important to understand as native VLANs are assigned on an interface-to-interface basis. 

Final episode of VLANs, here is the epilogue since most of the meat was covered in the previous two videos. In the last installment, we will cover layer 3 switches also known as multi-layer switches, network symbol is:
![[Pasted image 20250708144744.png]]

A multiplayer switch is layer 3 aware, meaning it is capable of switching and routing;
- You can assign an IP address to an interface, just like a router
- You can create virtual interfaces for each VLAN and assign IP addresses to those interfaces.
- You can configure routes on the multi-layer switch.
- It can also be used for inter-VLAN routing

In our earlier example, all inter-VLAN traffic goes to the router through the connection between SW2 and R1; in a small network, this is fine but all inter-VLAN traffic being routed through one point-to-point connection could cause a lot of congestion for larger networks. A multi-layer switch can replace SW2 (see below):
![[Pasted image 20250708145256.png]]
Now the connection between SW2 and R1 is simply for traffic meant for the internet (or any other non-connected networks).

Layer 3 switches have Switch Virtual Interfaces or SVIs which can be assigned IP addresses, if the end hosts' default gateways are configured to be this virtual interface instead of the router, which would allow the layer 3 switch to route traffic to the proper VLAN.

Since the default gateways are configured to be interface ports, what happens if a packet is meant for other networks? It will simply be dropped in the current setup; to remedy this, another subnet is allocated for the SW2-R1 point to point connection and a default route is configured on the layer 3 switch that forwards all traffic not meant for any VLAN to that interface. Having done that, the R1 interface can be reverted back to its normal settings. i.e. the sub-interfaces are removed using the commands:
	**no interface g0/0.10 AND .20 AND .30**

You can then issue the command:
	**default interface g0/0**
To revert the interface back to its default settings.
## Some Cisco IOS commannndsss
To display current VLANs:
	**show vlan brief**
This will display a table of all VLANs' numbers, names, and associated interfaces, see below:
![[Pasted image 20250707135336.png]]
All interfaces are assigned to VLAN 1, the default VLAN, by default. There are other VLANs used for technologies not within the scope of the CCNA, these are VLANs: 1002, 1003, 1004, 1005. The aforementioned VLAN as well as VLAN 1 (the default VLAN) cannot be deleted.

To assign an interface to a VLAN, you must enter interface configuration mode (or a range if there are multiple interfaces) and then issue the command:
	**switchport mode access**
	**switchport access vlan 10**

The first command sets the interface(s) as an access port, an **access port** is a switchport which belongs to a single VLAN, and is usually connected to an end host like a PC.

The second command assigns the access port to its specific VLAN. If you are assigning an access port to a VLAN that does not exist, you will receive the following message:
	*% Access VLAN does not exist. Creating vlan <vlan number used\>*

You can manually create VLANs by issuing the command:
	**vlan \<vlan number>**
in global configuration mode. This same command is also used to access *VLAN configuration mode*. Within VLAN configuration mode, you can issue the command:
	**name \<insert descriptive VLAN name>**
To change the name of the VLAN from the default "VLAN\<VLAN NUMBER>"

Now moving onto trunk ports. To configure a trunk port, go to interface congratulation mode for the interface in question and run the command:
	**switchport mode trunk**
If you then encounter an error about trunk encapsulation being set to "auto", this is because there are some older systems that support ISL which means encapsulation encapsulation protocol isn't automatically set to 802.1Q, and as such, you have to manually set the encapsulation type by running:
	**switchport trunk encapsulation dot1q**
and then rerunning:
	**switchport mode trunk**

 By running the command shown below, you can display all the trunk ports on the switch:
	 **show interfaces trunk**
![[Pasted image 20250707182610.png]]

Mode: On means the interface was manually configured as a trunk port. There are cases where it will automatically be configured into a trunk port, more on that later. 

To configure which VLANs are allowed on the trunk you can navigate to interface configuration mode, then run the command:
	**switchport trunk allowed vlan <action\>**
You can run:
	**switchport trunk allowed vlan ?**
To display the options, they are:
- WORD: comma separated list of allowed vlans
- add
- all
- except
- none
- remove
All self explanatory.

It's good practice to change the native VLAN to an unused VLAN such as 1001. You can do so by running the command:
	**switchport trunk native vlan 1001**

After doing all the switch configuration, we need to do the same for the router. This method of inter-VLAN routing is called *Router on a Stick*.

On the router side, one interface will be divided into multiple logical sub-interfaces, to do so enter global configuration mode then enter the command:
	**interface \<interface type> \<interface ID>\.<VLAN number\>**
e.g.
	**interface g0/0.10**
This will automatically create a sub-interface called g0/0.10. **Note:** the number does not have to match as you configure that later but it is highly recommended that it does match for accessibility.

After entering the above command you will enter into sub-interface configuration mode, and you can then change encapsulation type to 802.1Q by running a similar command as the one before:
	**encapsulation dot1q 10**
This command specifies that the sub-interface receive packets encapsulated using the 802.1Q protocol and tagged by VLAN ID 10.

Afterwards, you can bind the IP address of the sub-interface the same way you normally would, run:
	**ip address A.B.C.D(interface's IP address) A.B.C.D(bitmask)**

**Note:** Make sure to issue the command **no shut** on the "parent" interface when creating the sub-interfaces. Issuing no shut on the sub-interfaces themselves will not do anything if the parent interface is still administratively down (which it is by default on Cisco router).

First off, more on native VLANs, just as a reminder, each trunk interface has a native VLAN that is set to 1 by default, un-tagged frames arriving at a trunk port will be assumed to belong to the native VLAN, and packets sent out on the native VLAN will not be tagged. This is helpful as it reduces the frame size (by 4 bytes since that is the size of the tag) and allows for higher performance (more frames per second). 

In order to utilize the native VLAN feature, you can configure your router sub-interface with the native VLAN ID so that it behaves like a switch trunk with a configured native VLAN. To do, you need to run the command in sub-interface configuration mode:
	**encapsulation dot1q <VLAN ID\> native**
This will bind the sub-interface to a VLAN ID as well as make that VLAN the native VLAN on the sub-interface. You can also simply bind the gateway's IP address to the sub-interface without performing the step above for the native VLAN but do it for the other VLANs, i.e. the configuration will be:
![[Pasted image 20250708142658.png]]
The first interface g0/0 is the native interface and is not configured to do tagging whilst the other two are.

**Note:** It is still better for security to pick an unused VLAN as the native VLAN.

On the switch side, in order to enable layer 3 routing on a switch, you must issue the command:
	**ip routing**
in global configuration mode. Then enter the interface configuration mode for the interface you want to bind to an IP address and issue the command
	**no switchport**
this makes the port not function like a switchport, unsurprisingly. Afterwards, you can assign it an IP address by using the same command as in a router:
	**ip address A.B.C.D A.B.C.D**
Afterwards, you have to assign a default route which you can do the same way you do it in a switch:
	**ip route 0.0.0.0 0.0.0.0 <IP address of router interface\>**

In order to allow for inter-VLAN routing, you need to create some SVIs, to do so, issue the command:
	**interface vlan10**
in global configuration mode and then issue the command
	**ip address A.B.C.D(network address of VLAN) A.B.C.D(Gateway mask)**

SVIs are shutdown by default which means you also have to issue the **no shut** command.