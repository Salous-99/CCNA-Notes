So apparently Rapid Spanning Tree is the protocol that is actually needed for the CCNA, not classic spanning tree which was covered in [[Spanning Tree Protocol]].

Rapid STP is needed because using classic STP, it could take up to 50 seconds for the network to converge to its final topology. Cisco has developed their own version of rapid STP called *Rapid per-VLAN Spanning Tree* or Rapid PVST+. It builds upon the IEEE 802.1w standard that is used for rapid STP.

Rapid STP works the same as STP in the following ways:
- Same rules for root bridge election
- Same rules for root port election
- Same rules for designated and non-designated port election
It does the following things differently:
- Different STP costs (see table below)
- Does not have a blocking or disabled state, instead they were both combined into one *Discarding* state. If a port is administratively disabled, it will be in this state, if a port blocks traffic to prevent layer 2 loops from forming, that is also a discarding state.
- Does not have a listening state at all
- Non-designated ports are handled differently.
- All switches originate and send out their own BPDUs from their designated port, not just the root bridge. 
![[Pasted image 20250720165641.png]]

We will first look at the non-designated role, it has been replaced with two disintct RSTP roles:
- Alternate port mode
- Backup port mode

A port designated as an alternate port is one that is in the discarding state that can immediately transition to the forwarding state (without any transitional states) and become the root port should the actual root port connection go down. This functionality works like *UplinkFast*; an optional feature that can be enabled in classic PVST+. It is enabled by default in Rapid PVST+.

There is another feature that is optional in classic STP but has been built into rapid PVST+ which is BackboneFast. This feature allows a switch to forcibly expire the max age timer on a switch in order to send BPDU frames if a network topology change is detected.

The other kind of non-designated port is a backup port, this is a discarding port that receives BPDUs from another interface on the same switch. This happens when two interface are connected to the same collision domain (via a hub for example) and allows the backup discarding port to immediately transition into a designated, and forwarding state. Since it is rare to find a hub or anything similar that could cause two switchports to be on the same collision domain, RSTP backup ports are rare.

Switches in RSTP send out their own BPDUs every 2 seconds (Hello BPDUs). The switches age the BPDU information much faster than in classic STP. Instead of waiting for 10 missing Hello messages (20 seconds total), RSTP considers a neighbor lost or disconnected if 3 hello BPDUs go missing (6 seconds). After losing a neighbor, an RSTP switch will then discard all MAC addresses learned from that interface as they are no longer accessible through that interface.

RSTP distinguishes between 3 kinds of connection types:
- Edge: interfaces connected to end hosts. (they have portfast enabled be default)
- Point-to-point: direct connection between two switches
- Shared: a connection to a hub; must operate in half-duplex mode.

