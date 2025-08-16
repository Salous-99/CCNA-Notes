Networking models categorize and provide a structure for networking protocols and standards. A networking protocol is a set of rules defining how network devices should work and communicate. 

Standards are useful as they allow different devices from different manufacturers to communicate with one another without the need to have the exact same implementation.
# OSI Model
OSI stands for *Open Systems Interconnection*. The OSI model is a conceptual model that categorizes and standardizes the different functions in a network and is divided into layers, 7 to be precise.
## Layer 7 - the application layer:
- closest to the end user
- interacts with software that needs to communicate over a network
- HTTP is an example of an application layer protocol, and something like Firefox would be the application in question.
- functions of layer 7 include identifying communication partners and synchronizing communication

Communication between two application layer software is referred to as *same-layer interaction* and is facilitated by *adjacent layer interaction* which is the process by which data travels down the protocol stack and is **encapsulated** by lower layers and **decapsulated** as it travels up the protocol stack on the receiving device and to the peer application.
## Layer 6 - the presentation layer
- "translates" the data between application format to network format before sending it to or receiving it from the network.
- an example function of the presentation layer is encryption and decryption
- it also translates between different application-layer formats
## Layer 5 - the session layer
- controls dialogues between communicating hosts
- establishes, manages, and terminates connections between local and remote applications, e.g. your web browser (local app) and YouTube's server (remote app)

Application developers work with the top 3 layers of the OSI model. In fact, the more prevalent and actually used model (TCP/IP) encapsulates all top 3 layers into one Application layer.
## Layer 4 -  the transport layer
- segments and reassembles data for communication between end hosts
- segmentation is required as packets travelling over the network can only be so large before causing transmission problems.
- provides **host-to-host** communication over a network and inter-process communication within the same host
## Layer 3 - the network layer
- provides connectivity between end hosts on different LANs
- provides logical addressing
- provides path selection between source and destination 
## Layer 2 - the data link layer
- provides node-to-node connectivity and data transfer, for example:
	- PC to switch
	- switch to router
	- router to router
- defines how data is formatted for transmission across physical mediums
- detects and potentially corrects errors in transmission
- uses a different addressing system than layer 3
## Layer 1 - The physical layer
- Defines physical characteristics of the medium used to transfer the data, e.g. voltage levels, max. transmission distances, connector types, cable specs.
- converts digital signals to electrical or electromagnetic ones before transmission 

At different points in the encapsulation, the data takes on a new name:
- application layer: data
- transport layer: segment
- network layer: packet
- data link layer: frame
However all of these formats can be referred to by a single name which is *Protocol Data Units* or **PDU**s. A physical layer PDU is simply a bit. On bit travelling over the wire.
# TCP/IP Suite
The TCP/IP model's application layer encapsulates the top 3 layers of the OSI model, but when we talk about network layers, using the OSI model is more descriptive. Furthermore, the Link layer of the TCP/IP model encapsulates the data link and physical layers of the OSI model. 
![[Pasted image 20250624162044.png]]
