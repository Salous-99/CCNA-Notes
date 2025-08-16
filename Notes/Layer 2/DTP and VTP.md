Dynamic Trunking Protocol and Virtual Trunking Protocol. These are two Cisco proprietary algorithms used in inter-VLAN routing. These two protocols were **removed** from the CCNA exam material but knowing about them is still a good idea.

DTP is used to automatically configure port to either be an access or trunk port. One of the options when picking a switchport mode is to pick *dynamic* and select either *dynamic desirable* or *dynamic auto*. These modes do the following:
- a port configured in *dynamic desirable* mode will actively try to form a trunk and only configure itself to become an access port when connected to a port that is manually configured to be an access port.
- a port configured in *dynamic auto* mode will not actively try to form a trunk but will configure itself into a trunk if the other connected device wants to form a trunk.
In table format:

| switchport mode | auto   | desirable | trunk        | access       |
| --------------- | ------ | --------- | ------------ | ------------ |
| **auto**        | access | trunk     | trunk        | access       |
| **desirable**   | trunk  | trunk     | trunk        | access       |
| **trunk**       | trunk  | trunk     | trunk        | **mismatch** |
| **access**      | access | access    | **mismatch** | access       |
**Note:** DTP will not form a trunk with a router or a PC, etc... If you need to configure ROAS, you will need to manually set the port to trunk.

Aside from port administrative mode, DTP also negotiates the encapsulation protocol used for inter-VLAN routing. The two options are typically ISL and 802.1Q but newer Cisco products only have 802.1Q. The default setting on the switch is to negotiate for encapsulation as well.

On older Cisco switches, the default is ISL, however the industry standard nowadays is 802.1Q.

**Note:** To look at the administrative mode of a port, you can issue the command:
	**show interfaces <interface\> switchport**

VLAN Trunking Protocol allows for the automatic configuration of VLANs on one central switch that other switches can synchronize to. Older Cisco switches only supported versions 1 and 2, but newer versions support version 3.

Cisco switches using VTP can operate in one of three modes: **server**, **client**, and **transparent**. The default mode is server.

VTP servers can add, delete, and modify VLANs, storing this VLAN database in NVRAM. Each modification done to the database will increase it's *revision number*. The database with the highest revision number is considered the most up-to-date and other VTP clients and even other VTP servers will synchronize to it. VTP server switches send advertisements for their databases and associated revision number over their trunk interfaces (not access ports).

VTP clients cannot modify the VLAN database, and store this VLAN database in regular RAM for v1, and v2, and in NVRAM for v3. Clients will synchronize their database with the database of a server with the highest revision number, they will advertise their VLAN database and forward VTP advertisements to other clients over their trunk ports.

Run the command:
	**show vtp status**
to display information about the VTP mode on the switch. First, you need to create a VTP domain which you can do simply by issuing the command:
	**vtp domain <domain name e.g. cisco\>**

If you were to then create a VLAN on this switch, delete a preexisting VLAN, or do any other kind of modification, the revision number will increase an an advertisement will be sent out. Other connected switches, if their VTP domain name is still at its default value (null) will automatically become a part of the first VTP domain to advertise. If the switch receiving the advertisement is part of a different VTP domain it will **drop the frame????**

If you connect an older switch back to your network, it could cause (if VTP is enabled) other switches to synchronize with this older switch that has deprecated databases. This is one possible danger of VTP.

Finally there is VTP mode *transparent*, which will have the switch maintain its own database within NVRAM, as well as modify it freely. However, it will not synchronize its database to one with a higher revision number, it will ignore those advertisements and only *forward them if they are **within the same VTP domain***.