Packets coming from layer 3 to layer 2 are encapsulated by an Ethernet header and trailer.  The header contains the following fields:
![[Pasted image 20250625181819.png]]
- Preamble: 7 bytes long. Consists of alternating 1s and 0s. Used for synchronization of clocks between sender and receiver. **(Not usually included with the header if viewed through a packet sniffer)**
- Start Frame Delimiter or SFD: used to indicate the beginning of the Ethernet frame. It is one byte which is 10101011. **(Not usually included with the header if viewed through a packet sniffer)**
- Destination MAC address (receiver's MAC address)
- Source MAC address (sender's MAC address)
- Type: protocol type of the encapsulated packet (usually IPv4 or IPv6). **Note:** this field is sometimes used for the length of the frame, it depends on the Ethernet version used. Some things to note:
	- this field is 2 bytes long
	- if the contained value is less than or equal to 1500, the value indicates the length of the frame
	- if the contained value is 1536 or greater, it indicates the type of layer 3 protocol.

The Ethernet trailer has only one field which is the Frame Check Sequence or FCS: it is used by the receiver to verify whether or not an error occurred during transmission. This field is 4 bytes in length and can be used to detect corrupted packets by running a Cyclic Redundancy Check or CRC algorithm which takes the frame as input and generates the FCS, if the receiver generated FCS matches the received FCS, no errors occurred.

More over the length of a frame; the minimum size of a frame (header + payload + trailer) is 64 bytes. Removing the 18 bytes needed for the header (not including preamble and SFD) from the minimum size we conclude that the smallest payload size is 46 bytes. If a frame is sent with less data bytes than 46, zero bytes are added as padding to make up the difference. 

Some information about MAC addresses:
- 6 bytes long
- unique to device
	- globally unique mostly;
	- some MAC addresses or locally unique only
- First 3 bytes define the Organizational Unique Identifier or OUI which is unique to an organization
- The last 3 bytes are unique to the device itself
- Expressed as 12 hexadecimal characters

Consider the following simple LAN:
![[Pasted image 20250625193048.png]]

The switch SW1 has an internal MAC address table that holds key-pair value with the key being a MAC address and the value being the associated interface (the switch has 3 ports that are connected to PCs). This MAC address table is dynamically updated as the switch receives frames from different hosts, let us consider the behavior of the switch when:
- Switch receives a Unicast frame (a frame meant for one destination) from PC1 that is headed to PC2:
	- Switch adds the source MAC address of the frame (PC1's MAC address) to the table
	- Switch looks for MAC address of PC2 only to not find it, thus the received frame is an *unknown unicast frame*
	- Switch floods the unicast frame to all interfaces except F0/1 (because that is where it received the initial frame from)
	- **Note**: the switch's job ends here, but what happens after is both PC2 and 3 receive the frame, PC3 drops the frame since the destination MAC address does not match its own MAC address but PC2 receives it successfully
- Switch then receives a unicast frame from PC2 to PC1:
	- Switch adds the source MAC address of the frame (PC2's MAC address) to the table
	- Switch looks for MAC address of PC1 and fetches the corresponding interface, making this a known unicast frame
	- Switch forwards the frame on interface F0/1
- **Note**: The switch will remove dynamic MAC addresses (the ones it learns automatically from receiving frames) from its internal MAC address table after **5 minutes** of inactivity from that MAC address
- **Note**: The switch can have MAC addresses that are manually configured in it.

Address Resolution Protocol or ARP is used to resolve an IP address into a MAC address, Given that the host sending the ARP request doesn't know the MAC address of the receiver host, it sends out a frame with the destination MAC address set to the broadcast address. *Nothing new mentioned here*

## Some Cisco IOS commands
On a node with an arp table, you can run
	**show arp**
to see the arp table of the device

On a switch, you can view the MAC address table using:
	**show mac address-table**
You can clear all of its dynamically learned addresses by using
	**clear mac address-table dynamic**
You can clear a specific mapping using
	**clear mac address-table dynamic address \<insert MAC address here\>**
You can clear all addresses associated with an interface using
	**clear mac address-table dynamic interface <insert interface ID here\>**

You can ping different hosts using **ping** as you would in linux or windows. You can specify the size of the ECHO request message by running
	**ping \<IP address> size \<size in bytes of the ping message>**

