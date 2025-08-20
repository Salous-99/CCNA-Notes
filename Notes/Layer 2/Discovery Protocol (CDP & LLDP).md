Cisco Discovery Protocol and Link Layer Discovery Protocol are two *layer 2 discovery protocols* that are used to share information with and discover information about neighboring devices. The shared information includes: host names, IP addresses, device types, etc...

CDP is Cisco proprietary, LLDP is an industry standard (IEEE 802.1 AB).

Because these protocol give information about the devices, they can be considered security risks and are not enabled unless the network engineer makes the decision to enable them.
## CDP
CDP **\*** is enabled on Cisco devices by default. CDP messages are sent every 60 seconds from any interface in the up state to all connected neighbors on multicast MAC address: 0100:0CCC:CCCC. Devices receiving the CDP messages will process them then discard them. *They will **not** forward them to other connected devices*. Thus, only connected devices can become CDP neighbors. If a device does not receive any CDP messages from its neighbor for 180 seconds, it removes the CDP neighbor entry from its memory.

**\*Note:** CDP v2 is enabled by default. There is CDP v1 but it is not used as v2 offers more functionalities like identifying native VLAN mismatches for example.
## LLDP
LLDP is usually disabled on Cisco devices by default. Devices can run both CDP and LLDP at the same time. LLDP messages are sent once every 30 seconds, and the hold timre is 120 seconds. The messages are sent to multicast MAC address of 0180:C200:000E. LLDP also has an additional *re-initialization timer* that is set to 2 seconds by default and dictates how long the device will wait before initializing the connection with an LLDP neighbor.
# Cisco IOS
## CDP
CDP is enabled by default on Cisco devices and is active on all interfaces in the up state. To enable CDP, run the command
	**cdp run**
in global configuration mode, to disable it:
	**no cdp run**
To enable CDP on a specific interface, run
	**cdp enable**
within the interface's configuration mode or:
	**no cdp enable**
to disable it.

You can configure the timers using:
	**cdp timer <seconds\>** to set the CDP timer (before message are sent)
	**cdp holdtime <seconds\>** to set the CDP hold timer (before entries are dropped)

You can also disable v2 of CDP (this automatically enables v1 if CDP is still enabled) using:
	**no cdp advertise-v2**
### Show Commands
You can display information about the CDP process by running:
	**show cdp**
This will display the timer values, as well as CDP version running on the device.

You can display statistics on CDP traffic by running:
	**show cdp traffic**
This will show you the total numbers of messages sent and received, as well as other metadata about this traffic (for example number and type of errors that occurred during processing).

To show CDP configuration on each interface:
	**show cdp interface**
This will display the timers again, as well as the Ethernet encapsulation type used for that interface which is ARPA (also known as Ethernet 2).

To display neighbor information:
	**show cdp neighbors**
This will display information about the connected devices, this information includes:
- Device ID (host name)
- Interface connected to the device
- hold time left before entry is dropped (reset when the device receives a CDP message)
- Capability: identifies what kind of device is connected:
	- R - router
	- S - switch
	- I - IGMP (multicast related and beyond CCNA)
	- etc...
- Platform: model of the neighboring device
- Port ID of the connected port on the neighboring device (local interface is for this device, port ID is for neighboring device)

To display more details about the neighbors:
	**show cdp neighbors detail**
 This command will additionally display:
- software version
- VTP management Domain (LLDP can't do this)
- Native VLAN information	
- Duplex of connection

To view a single entry belonging to a single connected neighbor, run:
	**show cdp entry <neighbor's host name\>**
## LLDP
To enable LLDP globally, run:
	**lldp run**
in global configuration mode. Then you can enable it on each interface using:
	**lldp transmit** AND
	**lldp receive**
This allows LLDP to send and receive messages. To configure the timers:
	**lldp timer <seconds\>**
	**lldp holdtime <seconds\>**
	**lldp reinit <seconds\>**
### Show commands
The commands are very similar to their CDP counterparts:
	**show lldp**
	**show lldp traffic**
	**show lldp interface**
	**show lldp neighbors**
	**show lldp neighbors detail**
	**show lldp entry <neighbor host name\>**