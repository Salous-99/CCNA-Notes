Unlike routers, switches are not administratively down by default, and as such their default status and protocol are simply set to "down" and "down" and they become "up" and "up" once you connect a device to the switch.

![[Pasted image 20250629163530.png]]

The above command (**show interfaces status**) displays information about each port on the switch. Going across the list, we have:
- Port: port name/number
- Name: admin assigned description
- Status: connected or not
- VLAN: more on this later an another video, but if there are no virtual LANs, expect most of the numbers to be 1, with the exception of one interface that acts as the trunk (connecting multiple VLANs)
- Duplex: set to "auto" by default meaning the switch will automatically negotiate with the connected device for full-duplex communication and settle for half-duplex.
- Speed: bandwidth of the connection, a-100 means 100Mbps. Also set to "auto" by default and negotiated by the switch. 
- Type: Refer to [[Physical Cables and Interfaces]] for different speeds of different interface types.

The default "auto" setting is good at negotiating the best possible connection, but you can still configure an interface manually by doing the following:

- Enter global configuration mode
- issue the command
	  **int <interface name\>**
	For example:
		**int f0/1**
	You will then enter interface configuration mode as indicated by the *(config-if)#* line.
- To configure speed, issue
	  **speed ?** to display the possible speed settings and then pick one, e.g.
	  **speed 100**
- To configure duplex (half or full);
	  **duplex half** OR **duplex full**
	But mostly full, half duplex is just worse.
- To set a description (shows up as *Name* in the interface status menu);
	  **description <your description\>**

When configuring ports on a switch, it's good, security wise, to disable all the unused and unconnected interfaces; remember that switch ports are not administratively down by default, this means that they must be disabled manually. Luckily, you do not have to configure each one individually because Cisco IOS offers this handy command:
	**interface *range* f0/A - B**
This switch would for example have interfaces named f0/1, f0/2, ..., f0/N, where A < B < N. By using this command you can configure the entire range of interfaces from interface f0/A to f0/B simultaneously, allowing you to then set a description to show that this is an unused port, and disable all of the ports by issuing the **shutdown** command.

The **range** command also accepts a comma separated list as arguments. So, instead of just one range, you can provide multiple, for example if you wanted to re enable ports 5, 6, 11, 12, 15-20, you can use:
	**interface range f0/5-6, f0/11-12, f0/15-20**

# Collision Domains and Half-Duplex Switching
We mostly use full-duplex communication in modern day because it's faster and easier. In the olden days, instead of switches, there were things called Hubs, the network symbol for hubs is shown here:
![[Pasted image 20250630123926.png]]
It is used in an identical fashion as a switch and connects all hosts in a LAN. However, instead of forwarding frames to specific hosts, a hub acts as a repeater, it floods every frame it receives (like a broadcast frame) to all other hosts and the host with the matching MAC address would accept the frame and the other hosts would automatically drop it. 

The hosts connected within this LAN are in a *Collision Domain* a *"collision"* occurs when two hosts send a frame at the same time and the hub repeats both messages causing the two signal to interfere with one another making both of them illegible. Thus when half-duplex communication is needed, the connected hosts rely on *CSMA/CD* which stands for *Carrier Sense Multiple Access with Collision Detection*. The CSMA/CD protocol operates as follows:
- Before sending frames, devices listen to the collision domain (other connected hosts within the LAN) until they detect that other devices are not sending.
- If a collision does occur, the devices send a jamming signal to inform the other devices that a collision has happened.
- When a collision is detected, each device will wait a random amount of time before attempting to send again.

This makes a hub a *layer 1* device, as it does not process a MAC address, it simply repeats a signal. A switch solves this problem by allowing hosts to send and receive data at the same time and separating the connections between the different hosts. Collisions can still occur on switches but are typically the result of misconfiguration. Also, there is a chance that auto-negotiation is disabled on the connected devices which could result in a *Duplex mismatch* which occurs when one device is using half-duplex to communicate and the other device is using full-duplex.

The auto-negotiation process, if enabled on both the switch and device, will pick full-duplex and the fastest possible speed, if auto-negotiation is disabled on the connected device, the switch will try to "sense" the speed of the interface, if it fails to do so it it will pick the lowest speed available. After picking the speed, a Cisco switch will default to using half-duplex for 10 and 100 Mbps and full-duplex for Gigabit and faster.

Consider the following situation:
![[Pasted image 20250630125935.png]]

In this situation, all connected devices have disabled auto-negotiation. The switch can detect the speeds of the interfaces and accordingly, it configures the interfaces like so:
- G0/1:
	- detected speed: 10Mbps
	- chosen duplex: half-duplex
- G0/2:
	- detected speed: 100 Mbps
	- chosen duplex: half-duplex
- G0/3:
	- detected speed: 1000 Mbps
	- chosen duplex: full-duplex

The switch correctly configured interfaces G0/1 and 3 but caused a duplex mismatch on interface G0/2 which will cause collisions to occur which reduces the overall performance drastically. It is best to always enable auto-negotiation to avoid situations like these.

**The auto-negotiation process happens in the same way on routers**
## Interface errors
When displaying an interface using **interfaces show <interface name\>**, some statistics will be displayed at the bottom:
![[Pasted image 20250630130855.png]]
You do not need to know all of them;
- packets input and bytes : packets received on the interface and the total number of bytes in those received packets
- received broadcasts: broadcast frames incoming from the device to the switch
- runts: frames smaller than the minimum frame size of 64 bytes.
- giants: frames larger than the maximum frame size of 1518 btyes.
- CRC: frames that failed their CRC check (FCS Ethernet trailer).
- frame: frames that have an incorrect or illegal format
- input errors: total errors in incident frames.
- Output errors: total errors in outgoing frames.
- packets output and bytes : packets sent from the interface and the total number of bytes in those sent packets.

