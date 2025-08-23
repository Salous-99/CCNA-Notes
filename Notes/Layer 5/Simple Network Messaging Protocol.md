SNMP is more than just a simple protocol, it is closer to a framework for managing devices within a network. SNMP v1, for example, included RFC 1065, 1066, and 1067. Updated RFCs constituted later versions of SNMP.

SNMP has two main types of devices:
- Managed devices:
	- devices being managed by SNMP
	- examples include routers and switches
- Network Management Stations (NMS)
	- Devices being used to manage the managed devices
	- These act as SNMP "servers". The usage of quotes is because they are usually referred to as an NMS.

Consider the following network:
![[Pasted image 20250823141832.png]]
SNMP can afford the following capabilities:
1. Managed devices can notify the NMS of any events. *for example if the G 0/1 interface of SW1 goes down, the switch can inform the NMS server*
2. The NMS can ask the managed devices about their current status and configuration. *for example the NMS can query R1 to see the state of its G 0/1 interface*
3. The NMS can tweak the configurations of managed devices. *for example, the NMS can inform R1 to change the IP address bound to G 0/1 to something else*

![[Pasted image 20250823143330.png]]
The following diagram shows an oversimplified structure of SNMP. The NMS itself is not always a "server" in the sense that it sits in a cold data center and is operating from some rack somewhere. The NMS is usually the network admin's laptop or device that is running an SNMP application that is used as an interface. This application displays data about the devices using visual aids and other information relaying methods. It also provides an interface that can be used to request data and update configuration options on the managed devices. The SNMP manager is another component of the SNMP software that is used to actually communicate with SNMP agents running on the SNMP managed devices. The manager and agents are responsible for communication, the SNMP manager is responsible for centralizing this information as well as issuing new configurations for the devices. When these configurations are issued and received by the clients' SNMP agents, they are then processed by the Management Information Base or MIB. This MIB contains individual variables that are related to the operation of the managed device, for a switch or a router, there are variables that define the status of each interface, device temperature, CPU usage, etc... Each of these variables is identified by an *Object Identifier* or OID. These OIDs are stored in the MIB.
### Moreover OIDs
These identifiers are not assigned manually but are standardized. You can refer to [this website](https://oid-base.com/) for OID identifying capabilities. Not much more is required for the CCNA aside from knowing that OIDs exist and are used for SNMP.
### SNMP versions
There are three main versions of SNMP that are widely used:
1. SNMP v1: most common and the one we are learning about now
2. SNMP v2 c: allows an NMS to request large amounts of data in a single request, thus increasing efficiency. SNMP v2 alone removed the "community strings" (used as passwords) feature from SNMP v1, but were added back in SNMP v2 **c**.
3. SNMP v3: uses strong encryption and authentication. Preferred version.
**Note:** where possible, it is best to use SNMP v3. SNMP v2 "community strings" are not a powerful safety feature as they are sent over the network in plaintext. SNMP v3 encrypts them.
## SNMP Messages
![[Pasted image 20250823145120.png]]
Read messages; sent from the NMS to the managed device:
- Get message: used to request a certain variable with a given OID.
- GetNext message: used to discover the available variables in the device's MIB
- GetBulk message: like GetNext but more efficient since it can request a larger list of variables (introduced in SNMP v2)
Write messages, sent from the NMS to the managed device:
- Set message: used to update the value of a certain variable with a given OID.
Notification, sent from the managed device to the NMS:
- Trap message: sent as a response to an event. It also goes unacknowledged by the NMS
- Inform message: sent as a response to an event. Is acknowledged by the NMS. Originally used for communication between multiple SNMP managers but in later versions, clients were also allowed to send inform messages to the NMS.
Response, sent in response to receiving a message, can be sent in either direction:
- Response message: sent as an answer to a received SNMP message.

**Note:** SNMP Managers listen for messages on UDP/162.
	  SNMP Agents listen for messages on UDP/161
## Cisco IOS Configuration
This is **not needed** for the CCNA, but here are some configuration options anyway; from global configuration mode, you can run the following commands to designate a network device as an SNMP client (setting up an NMS is too complicated).

Run the following *optional* commands to set some contact information:
	**snmp-server contact <email address\>**
	**snmp-server location <location name\>**

You can configure a community string to use as a makeshift password using:
	**snmp-server community <password\> {ro (read only) | rw (read and write)}**
This creates a "password" that allows an NMS to either only read variables, or read and modify values.

Then run the following command to bind an SNMP client to its NMS:
	**snmp-server host <IP address of NMS\> version <version type 1/2/2c\> <community string\>**

You can now enable certain traps to inform the NMS of specific changes, here are some examples:
	**snmp-server enable traps snmp linkdown linkup**
	**snmp-server enable traps config**

The first command makes the router send trap messages to the NMS in case a link goes up or down. The second command tells the router to send trap message if the configuration on the router changes.