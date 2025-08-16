Before discussing this protocol, first we will discuss redundancy. It is important to maintain up-time of business critical infrastructure by adding redundancy; It is an essential part of network design. As much as possible, redundancy must be implemented at every point in the network.

Consider the following network:
![[Pasted image 20250716183554.png]]
This network is considered to be poorly designed due to the existence of multiple points of failure. There is only one connection to the internet which means if that connection went down (due to a service interruption or problem with the router, the entire network loses internet connectivity, if the core switch connected to the router fails, the entire network goes down, and many more.

This is a better design:
![[Pasted image 20250716183812.png]]
Two connections to the internet (more costly sure but more reliable), multiple connections between switches in case any one switch fails, etc...

**Note:** most PCs only have a single network interface card which means it is difficult to provide them with redundancy. Servers typically do have multiple NICs for that purpose.

Spanning tree protocol is a layer 2 protocol, it is used to provide the redundant switches with the routes for the frames to travel through. There is a problem with the above configuration, it is the existence of switching loops.

Consider the following setup:
![[Pasted image 20250716184405.png]]
What happens if PC1 sends a broadcast? Or if any switch receives an unknown unicast frame that it needs to flood to the network?

If SW1 receives a broadcast frame and floods it, switches 2 and 3 will receives it and flood it as well, out of all interfaces except the one they received it on, so that means a broadcast frame sent to: PCs 2 and 3, and switches 2 and 3. Switches 2 and 3 will then repeat the same process, broadcasting th
In most situations, we enable portfast only on ports that connect to end hosts (such as PCs) there are two examples where it makes sense to use portfast on a trunke frames to SW1 (twice), PCs 2 and 3. SW1 will receive the same broadcast it sent out. This goes on forever and takes up all your bandwidth. There is also another problem known as *MAC address flapping* which occurs when a switch continuously changes the mapping of a MAC address to different interfaces as it receives frames from that MAC address on different ports.

Spanning tree protocol is an IEEE standard number 802.1D. Switches from all vendors run STP by default, not just Cisco. STP prevents later 2 loops by placing redundant ports in a blocking state, essentially disabling the interface. These interfaces act as a backup and enter the forwarding state if an active interface fails. Interfaces in the blocking state will not forward any frames but will still continue to send and receive STP messages called *Bridge Protocol Data Units* or BPDUs.

Here is how the protocol works:
- Switchports are set as either forwarding or blocking, and STP creates a single path to and from each point in the network.
- STP enabled switches will send and receive HELLO BPDUs on all interfaces every 2 seconds
- If a switch receives a Hello BPDU on an interface, it then knows that there's another switch there. (PCs and routers do not use STP and do not send Hello BPDUs)

The format of an STP BPDU is:
![[Pasted image 20250716190552.png]]

A bridge is an older network device that was the short-lived predecessor of the switch. When you see bridge, think switch.

Switches use the bridge ID to elect a *root bridge*; all ports of the root bridge are in the forwarding state and all other switches must have a path to the root bridge.

For a switch to become the root bridge it must have the **lowest** bridge ID. First the bridge priority is compared, which, by default is set to 32768. If the priority matches, the MAC addresses are compared and the lowest MAC address becomes the root bridge.

The Bridge priority field has been updated and split into two parts;
- 4-bit bridge priority
- 12-bit VLAN ID (also known as the Extended System ID)
![[Pasted image 20250716192052.png]]
Given this new setup, if you aim to increase or decrease the bridge ID without changing the VLAN ID, you can only do so in increments of 4096 due to the math of it all.

The reason for this update is Cisco's usage of a separate version of STP called PVST (Per-VLAN Spanning Tree) which runs separate instances of STP for each VLAN. This means that within each VLAN, interfaces can be forwarding or blocking. More on this later.

When a switch is powered on, it assumes that it is the root bridge and starts sending out BPDUs. If it receives a superior BPDU with a lower bridge ID, it will give up and stop forwarding BPDUs. Once the network converges and all switches agree on the root bridge, only the root bridge will send BPDUs, other switches will forward these BPDUs but will not generate their own BPDUs.

Each of the other, non root bridge switches will select one port to be the **root port**, this port is the one associated with the lowest **root cost**, and will be set to a forwarding state. To understand the cost, we need to first learn some metrics about the STP overhead, see table below:
![[Pasted image 20250716192439.png]]

The root cost is the total STP cost (interface dependent) of each interface the frame has to travel through to get to the root bridge. By default, each port on the root bridge itself has a root cost of 0.

The STP messages sent out by the root bridge will advertise its root cost of 0 on all interfaces, other switches will then add their own cost of sending and then forward that BPDU on all interfaces,

for an example, refer to the network below:
![[Pasted image 20250716184405.png]]
SW1 is the root bridge and all switches are connected by 1 Gbps interfaces incurring a cost of 4 when sending, SW1 will advertise its cost of 0 to SW2 and 3, SW2 and 3 will add 4 to 0 and forward that advertisement to each other, then each switch compares the cost of sending to SW1 directly via the direct connection with a total cost of 4 or indirectly via the other switch for a total cost of 8. In this case it is obvious that the direct connection is better. If the cost is equal, a switch picks a root port depending on the bridge ID of the neighboring switches, it will select the port going to the switch with the lower ID. The last tie breaker needed is for situations where you have two connections between two switches this means the cost and bridge ID will be equal; if that is the case, the port connected to the neighbor's lower port ID will be designated as the root port.  

Following these steps, we should arrive at the following:
![[Pasted image 20250716194252.png]]
The root bridge is SW1, all of its ports are forwarding ports, SW3 and SW2 have picked their root ports to be the ones with the lowest root cost, now it is time to actually block some ports to prevent a loop from happening.

A rule of STP is that there is a single STP designated (or forwarding) port, between SW1 and 2, G0/0 is the forwarding port and the other is the root port, between SW1 and 3, G0/1 is the forwarding port and the other is the root port.   

After applying the above steps to determine the root ports on all switches, all other collision domains must abide by the above rule so the final collision domain between SW2 and 3 must have one port be a designated port and the other one be non-designated which means in the blocking state.

To determine which port gets blocked and which one is in the forwarding state, follow these riles:
- the switch with the lowest root cost will have its port be the designated port
- if equal, the switch with the lowest bridge ID will make its port the designated port
- the other switch makes its port non-designated

Within the Cisco IOS, you can display information pertaining to STP by running, in privileged EXEC mode:
	**show spanning-tree** OR
	**show spanning-tree vlan <vlan number\>** to show the tree for a specific VLAN OR
	**show spanning-tree summary** for STP information about all VLANs 
	**show spanning-tree detail** for details about the root cost of each interface

This far, we have only considered two stable states that an STP enabled port can be in which are *forwarding* and *blocking*. There are two further transitional states that a port can be in which are *Listening* and *Learning*. There is also the *disabled* state which is when the interface itself is administratively disabled.  

Listening and learning are transitional states which are passed through when an interface is activated or when a blocking port must transition to a forwarding state due to a change in the network topology. 

Interfaces in the *Blocking* state:
- effectively disabled to prevent loops
- do not send nor receive regular network traffic
- still receive STP BPDUs which the switch can use to make decisions about potentially enabling a blocking port.
- do not forward STP BPDUs
- do not learn MAC addresses; if frames are received on a blocking port, the frame is dropped and the source MAC address is not added to the MAC address table.

Interfaces in the *Listening* state:
- if they are designated or root ports, they enter this state after the blocking state 
- are in this state for 15 seconds by default which is determined by the **Forward Delay** timer
- only receive and forward STP BPDUs but do not send any
- do not receive or send regular traffic
- do not learn new MAC addresses

Interfaces in the Learning state:
- enter this state after the listening state
- are in this state for 15 seconds by default, also as determined by the forward delay timer
- only send and receive STP BPDUs but not regular traffic
- Learns MAC addresses from regular incoming traffic frames.

Finally, interfaces in the forwarding state:
- Sends and receives STP BPDUs
- Sends and receives regular traffic
- Learns MAC addresses of incing frames

STP timers:
- Hello timer: how often the root bridge sends Hello BPDUs - 2 seconds by default
- Forward delay: how long will the switch stay in listening and learning mode - 15 seconds by default (**Note:** 15 seconds for listening and another 15 seconds for learning state meaning the overall delay between blocking and forwarding is 30 seconds)
- Max age: How long an interface will wait after ceasing to receive Hello BPDUs to change the STP topology - 20 seconds by default (10 * Hello timer)

**Note:** switches only sends STP BPDUs on their designated ports (root ports **not** included).

If a switchport does not receive any STP BPDUs for 20 seconds, it will then reevaluate its STP choices including: root bridge, designated, and non-designated port. If a non-designated port is selected to become a root or designated port, it will then enter the listening and then the learning stages. On the other hand, some interfaces might become non-designated, in that instance, the port can transition directly from forwarding to blocking without going through any transition phases.

STP BPDUs are sent to a specific MAC address, this MAC address is not the one belonging to the port, it is a special one associated with the Cisco PVST+ process. The MAC address is 01:00:0C:CC:CC:CD. The PVST process has a MAC address of 01:80:c2:00:00:00.

**Note:** PVST+ is a newer version of PVST that also supports 802.1Q encapsulation (whereas PVST only supports ISL).

The format of a STP BPDU is:
![[Pasted image 20250718112114.png]]
- Protocol identifier: set to 0x0000 to signify the spanning tree protocol
- Version: 0x00 for classic spanning tree
- BPDU/message type: 0x00 for configuration (any message where this is a non-zero value is probably beyond the scope)
- BPDU Flags: Used to signal topology changes
- Root identifier: Root bridge priority, system ID extension (VLAN) and MAC address.
- Root path cost: cost to get to root bridge
- Bridge identifier: bridge ID, system ID extension and MAC address of the sending switch
- Port identifier: port ID and number of the sending port on the sending switch
- Message age: starts at 0 at the root bridge and increases by 1 at each switch hop
- Max age
- Hello time
- Forward Delay
## STP Toolkit
First off, most of the STP configurations happen on a per-interface basis, and not for the whole switch. That said, to view the STP related configuration of any one interface, issue the command:
	**show spanning-tree interface <interface\> detail**
Within privileged exec mode.
## STP Portfast
solves one problem with STP; when a port becomes designated it goes through twice the forward delay timer in order to become a forwarding port; when this port leads to another switch, this is necessary to do, but if it is connected to an end host, there is no need to do any listening or learning and can connect immediately, this is done via *Portfast*. **Note:** this should only be enabled on ports connected to end hosts, the reason being that layer 2 loops are dangerous. To enable it you must enter the interface configuration mode for the interface(s) you want to configure then issue the command:
	**spanning-tree portfast**
You should be greeted with a warning message reminding you to only enable this feature for connected end hosts.

You can also issue the command:
	**spanning-tree portfast default**
To enable portfast on all ***access ports***. Note, doing this is a bad idea and could lead to unexpected layer 2 loops. In most situations, we enable portfast only on ports that connect to end host (such as PCs) there are two examples where it makes sense to use portfast on a trunk port instead of an access port and these are:
- Trunk ports connected to virtualization servers.
- Trunk ports connected to routers using a ROAS configuration. 
To configure port fast on a trunk port, you need to issue the command (per trunk interface):
	**spanning-tree portfast trunk**

To disable portfast, you can issue the command:
	**spanning-tree portfast disable**
in interface configuration mode.
## BPDU Guard
If an interface with BPDU guard enabled receives a BPDU from another switch, the interface will be shut down. To enable it, issue the command:
	**spanning-tree bpduguard enable**
From the interface configuration mode of the interface(s) you want to configure.

It is a good idea to enable BPDU guard on ports with portfast enabled to evade such situations where re-cabling could lead to switching loops, you can do this for all ports with portfast enabled by issuing the command:
	**spanning-tree portfast bpduguard default**
This will enable BPDU guard on all ports with portfast enabled by default. You can and should couple this with the aforementioned command:
	**spanning-tree portfast default**
ports disabled by BPDU guard can be re-enabled by being restarted; this can be achieved
by administratively disabling them (using **shutdown**) and re-enabling them (via **no shutdown**).

You can disable BPDU guard on interfaces it is enabled on by issuing the command
	**spanning-tree bpduguard disable**

Receiving a BPDU on a port with BPDU guard enabled will cause that port to enter a state of "error disabled" or err_disabled. A port in this state has to be reset manually by shutting off and turning back on administratively. You can do the recovery automatically however by enabling error disable recovery which you can issue the command:
	**errdisable recovery cause <insert cause (in this case: bpduguard\>**
in global configuration mode. Once enabled, if you were to issue the command:
	**show errdisable recovery**
You will be shown 0this table:
![[Pasted image 20250718153207.png]]
This means that error recovery is enabled for bpduguard violations which means the disabled interface will be re-enabled automatically. This will occur after 300 seconds as the timer interval shows, to change this time interval, issue the command:
	**errdisable recovery interval <time in seconds\>**
## BPDU Filter
A BPDU filter can be used to filter out any not needed BPDU messages that a switch sends; for example, BPDU guard can be enabled on the interface connected to an end host (PC) to prevent that port from sending any BPDUs; this makes sense as the PC does not need these messages, so they only take up the interface's bandwidth (minimal amount sure, but not zero) furthermore, it leaks some information about the network topology which is a security concern.

To enable the filter on a singular port, issue the command:
	**spanning-tree bpdufilter enable**
Within the interface configuration mode for that port. This will do the following:
- The port will not send any BPDUs
- The port will drop all BPDUs it receives
- Effectively, this disables STP on that port. (risky)

You can also enable BPDU filter by default from global configuration mode by issuing:
	**spanning-tree portfast bpdufilter default**
This will do the following:
- Enables BPDU filter on all portfast enabled interfaces
	- can be disabled on specific ports via **spanning-tree bpdufilter disable**
- Ports will  not send BPDUs
- Ports will still receive SPDUs
## Root Guard
If root guard is enabled on an interface, it will not accept BPDUs with a lower Bridge ID as BPDUs coming from the root bridge, the interface will simply be disabled instead. This allows for the network associate to maintain network topology and prevent against a situation where some entity, maliciously or not, plugs in a switch with a lower bridge ID into the network that could cause a layer 2 loop.

Why would it be important to enforce the root bridge placement? Why does it matter? What things should you consider when picking a root bridge? Here are some things:
- Optimal traffic flow
	- minimize latency
	- minimize congestion
- Stability and Reliability of switch
	- root bridges should be your newer model switches
	- root bridges should not fail so easily or quickly

Consider the following network:
![[Pasted image 20250719115152.png]]
The root bridge is switch 1; this ensures optimal traffic flow from switches 2 and 3 to R1, thus it is a good placement. If switch 3 was the root bridge instead, outgoing traffic from switch 2 that wants to reach either the internet or the other VLAN must pass through switch 3, taking up more bandwidth as well as cause congestion on the link between switches 1 and 3.

To enable root guard on an interface (cannot be enabled by default in global config mode), issue the command:
	**spanning-tree guard root**
from the interface configuration mode of the interface in question.

Once an interface with root guard enabled receives a superior BPDU, it enters into a "broken" state effectively shutting the interface down. To fix this, first you must fix the root problem that cause the issue and then wait for 20 seconds (value of the max age timer) and the disabled ports will be re-enabled automatically.
## Loop Guard
If loop guard is enabled on an interface, even it that interface stops receiving BPDUs, it will not start forwarding, it will be disabled instead. This helps prevent against failures in one direction on certain links (called a unidirectional link).

These scenarios are common when using fiber optic cables that typically consist of two fibers, one for transmitting and the other for receiving. If one of them becomes damaged, as fiber optic cables are wanting to do, you get a unidirectional link. Typically, a unidirectional link is detected by the switch and the link is immediately disabled, but sometimes, the damage in the link is not detected and you get a situation where two interfaces are in the up/up state even though one switch cannot hear the other. Let us now examine a situation in which this causes a problem. Consider the following diagram:
![[Pasted image 20250719122524.png]]

Root bridge is switch 1, and the link between SW2 and 3 is unidirectional; SW2 cannot send frames to SW3, but SW3 can send frames to SW2. Previously (before the damage to the link occurred) SW3 used to receive BPDUs from SW2 (forwarded from SW1) that maintained the topology, after the error occurs, and after the max age timer has elapsed, the G0/1 interface on SW3 will assume that there is not a loop anymore (as evidenced by not receiving any BPDUs) and will start forwarding frames out of its G0/1 interface to SW2,  and we will have a layer 2 loop on our hand, this loop will never rectify because switch 2 will ignore BPDUs from switch 3 because they are inferior. Since we now have a loop, we are prone to broadcast storms.

Common practice is to enable loop guard on non-designated and root interfaces (effectively interfaces that are supposed to receive BPDUs). To enable on a specific interface, issue the command:
	**spanning-tree guard loop**
within interface configuration mode or enable it by default on all ports by
	**spanning-tree loopguard default**
and disabling it on specific interfaces using
	**spanning-tree guard none**

**Note:** Loop guard and root guard are mutually exclusive and serve opposite purposes, also, if loopguard is enabled on a root guard enabled interface, root guard will be disabled and vice versa.
## STP Load Balancing
PVST allows for a separate STP instance for each VLAN, the extra complexity allows for more efficient usage of the switchports' bandwidth as different switches, connected to different VLANs could all act as root bridges for different VLANs allowing traffic to be split among all switches instead of being concentrated in the root switch.     
## Cisco IOS Commands
To configure a switch to become the root bridge, you can issue the command:
	**spanning-tree vlan <VLAN number\> root primary**
in global configuration mode. The above command sets the bridge priority of the switch to 24576 or a value that is 4096 lower than the lowest bridge priority on the network if some switch has a bridge priority lower than 24576. In effect, this command is actually:
	**spanning-tree vlan <VLAN number\> priority 24576**

To set a secondary root bridge (second lowest priority), you can issue a very similar command:
	**spanning-tree vlan <VLAN number\> root secondary**
This command sets the priority to 4096 more than 24576 (primary bridge priority) which is 28672.

You can configure the cost of each port from the standard values (see table above) for more granular modification, to do this, you can issue the command (from interface configuration mode):
	**spanning-tree vlan <VLAN number\> cost <STP sending cost\>**
To change the cost, or:
	**spanning-tree vlan <VLAN number\> port-priority <port priority in increments of 32\>**
