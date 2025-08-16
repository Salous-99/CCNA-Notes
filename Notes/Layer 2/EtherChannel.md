![[Pasted image 20250722133304.png]]
Consider the above network; first off the distinction between ASW1 and DSW1 is as follows:
Access layer switches are for end hosts, distribution layer switches are for other access layer switches to connect to.

If there are 40+ hosts connected to the access layer switch and all of these hosts are trying to connect to the internet, there will be a lot of congestion on the link between ASW1 and DSW1. An admin might decide to do something stupid like:
![[Pasted image 20250722133509.png]]
To try and remedy the problem but this does not result in any improvement. Why does that happen? Because STP zaps all of those extra links because as far as the STP process is concerned, layer 2 loops are a big no-no. See below:
![[Pasted image 20250722133707.png]]
ASW1 for sure is not the root bridge, or even a high (technically I mean low) priority bridge, STP will see those extra interfaces and assign them a alternate, discarding port state and status, and all traffic will still go through one connection.

Enter EtherChannel. This feature allows us to group together many physical interfaces into one logical interface that STP recognizes as a single link. In network diagram, an EtherChannel is illustrated like so:
![[Pasted image 20250722134017.png]]
EtherChannel basically allows us to configure load balancing on network interfaces. The algorithm that is used to pick the physical interface has the following inputs:
- Source MAC address
- Destination MAC address
- Both source and destination MAC addresses
- Source IP address
- Destination IP address
- Both source and destination IP addresses
For example; all traffic meant for destination MAC address <address of some important internal server\> goes through physical interface G X/Y. Some older models do not support filtering based on IP addresses and only support MAC addresses. Advanced switches can also filter based on layer 4 protocols.

To view the current mode of load balancing, use the command:
	**show etherchannel load-balance**
To change it, you can use:
	**port-channel load-balance <mode\>**
(use **port-channel load-balance ?** to view all the available modes)

There are 3 methods by which to form an EtherChannel:
1. Static EtherChannel: statically configure interfaces to operate as one logical interface (usually avoided because you want the two switches to dynamically manage the channel)
2. PAgP or Port Aggregation Protocol:
	1. Cisco proprietary protocol
	2. Dynamically negotiates the creation and maintenance of the EtherChannel
3. LACP or Link Aggregation Control Protocol:
	1. Industry standard protocol, IEEE 802.3ad
	2. Dynamically negotiates the creation and maintenance of the EtherChannel

## PAgP/LACP Layer 2 EtherChannel Configuration
enter interface range configuration mode for the interfaces you want to group as an EtherChannel **\***, and enter the command:
	**channel-group \<group number> mode <port mode\>**
The group number is just a numerical identifier of the channel and can be assigned counting up from 1 if there are multiple channels, the mode can be one of the following:
- active: LACP
- passive: LACP if connected device can also use LACP
- desirable: PAgP
- auto: PAgP if connected device can also use PAgP
- on: EtherChannel only

**\*Note:** not needed, and interfaces can be configured separately if needed, but it is best to do it in range configuration mode as many settings must match in order for the EtherChannel to be established

When picking the mode, you must note that the settings on the connected switch have to also allow for an EtherChannel to form which means:

| Mode          | desirable | auto    | active  | passive | on      |
| ------------- | --------- | ------- | ------- | ------- | ------- |
| **desirable** | success   | success | No EC   | no EC   | no EC   |
| **auto**      | sucess    | no EC   | no EC   | no EC   | no EC   |
| **active**    | no EC     | no EC   | success | sucess  | no EC   |
| **passive**   | no EC     | no EC   | sucess  | no EC   | no EC   |
| **on**        | no EC     | no EC   | no EC   | no EC   | success |
You can also issue the command:
	**channel-protocol LACP** OR **channel-protocol PAgP**
To select the negotiation protocol for the EtherChannel although it is not needed if you select any mode other than *on*. It will also allow you to switch between active and passive or desirable and auto but that is about it.

Issuing this command on the switch will create a *port channel* with the number you also provided in the command. This will appear on the list of interfaces in the output of **show interface status**

Once the port channel is created, you can configure it (as you would configure any other interface) using:
	**interface port-channel <port channel number\>**

configuring anything on the port channel applies the configuration to all the individual interfaces. In fact member interfaces must have the following matching configurations:
- Same duplex
- Same speed
- Same switchport mode
- Same allowed VLANs and native VLAN (for trunk port)

If these configurations differ on any port, it will be excluded from the channel.

To verify the status of an EtherChannel, you can use the command:
	**show etherchannel summary**
 You can also use:
	 **show etherchannel port-channel**
for some more detail
## PAgP/LACP Layer 3 EtherChannel Configuration
Pretty much the same as the layer 2 stuff....just issue the **no switchport** command on the interface range you want to configure and follow the same steps. After creating the port channel, you can enter the interface configuration mode for that port channel to assign the IP address of the channel. STP will not operate on routed ports.