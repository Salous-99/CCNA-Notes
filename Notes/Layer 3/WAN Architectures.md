A WAN is a Wide Area Network, and it extends over a large geographic area. A WAN is used to connect geographically separate LANs. For example, an organization with separate offices in multiple cities has many disintct LANs, each one in a different city, the network connecting all these different LANs together is the WAN.

The internet as a whole can be considered a WAN but typically, the usage of the term refers to the private connections connecting different LANs belonging to one organization. Over the internet, different LANs can be connected through the creation of private WAN connections, this is what VPN is.

Commonly, different LANs belonging to one organization need to connect to a data center, the data center is not necessarily in the same location as any of the offices and so must be connected remotely through a *Leased Line* which is a connection only used by the organization to directly connect the office(s) to the data centers, it is not a shared line.
![[Pasted image 20250908114711.png]]
This is a typical star topology but in the context of a WAN, it is known as a **Hub-and-Spoke topology**. Important to note that whilst the leased lines do directly connect the sites to the data center, these lines still go through the local service provider. The assumption here is that the service provider covers a large enough area as to encompass all offices and the data center and can create these "leased lines". 

You can create the same network structure directly over the internet instead of going through the service provider (you will still go through your ISP no matter what, but "over the internet" here means that you are not signing an agreement with your ISP to lease any private connections) by utilizing VPN. VPN allows for the creation of an encrypted tunnel carrying data securely over the internet without worry that someone intercept the packet on the way will be able to access or modify the data.
## Leased Lines
- Dedicated physical link, typically used to connect two sites.
- Leased lines do not use Ethernet for layer 2 encapsulation, instead they use PPP or HDLC.
	- These are becoming less popular as fiber optic cables become more common and cheap
	- Fiber optics can be used to carry data quickly over larger distances, allowing for the usage of Ethernet encapsulation over PPP or HDLC whilst still having a relative low cost.
## Multi Protocol Label Switching
Short form: MPLS. Service providers have MPLS networks that they can use to offer connectivity solutions to large enterprises with multiple locations. MPLS relies on *label switching*, a label is added to a packet travelling to the ISP's MPLS network as it enters the network through the Provider Edge (PE) Router and that label is used to make routing decisions instead of the IP address. These labels can also be used to provide VPN services to customers.

The Customer Edge (CE) routers do not use MPLS, these are only used by the PE and Provider (P) routers. When using layer 3 MPLS VPN, the CE and PE routers peer using OSPF to share routing information, consider the network below:
![[Pasted image 20250908123704.png]]
In this network, the CE and PE routers will peer in order to share information via OSPF, the PE routers will add labels to these packets as they travel through the service provider's network to the other CE through another PE of the service provider. Through OSPF, routing information is shared between the two offices and the path taken through the service provider's network is considered a black box.

You can also use layer 2 MPLS VPN in which the CE and PE do not form peerings giving the impression that the two CE routers are directly connected and the ISP's network appears completely transparent, or as if it were just a switch connecting two devices within the same LAN. In fact, in the logical view of this network, the connection between the CE routers in offices A and B appear to be a point-to-point connection.

MPLS routing is used internally within the ISP's network only and does not care for how the information gets from the PE to the CE router, so a network like so: ![[Pasted image 20250908124222.png]]
is completely fine.
## Internet Connections
### Digital Subscriber Line
A DSL provides internet connectivity by utilizing the phone line installed in the home. For a DSL to work, it requires a device called a *Modem* (modulator-demodulator), the job of a modem is to convert from digital to analogue and back. Computers generate and are able to parse through digital information; over a wire, this looks like a series of high and low voltages representing the ones and zeros. Phone lines, however, carry an analog signal as such, you need a converter which is the modem, this modem is either built into the home router or stored as a separate device.
### Cable Internet
Cable internet utilizes the TV cable for internet connectivity, and just like a DSL, a modem is required.
## Redundant Internet Connections
Some terms to learn quickly:
- Single Homed connection: 1 connection to an ISP (least redundancy)
- Dual Homed connection: 2 connections to the **same ISP**
- Multi Homed connection: 2 connections to **different ISPs**; 1 connection for each ISP
- Dual Multi Homed connection: 2 connections to **different ISPs**; 2 connections for ISP (most redundancy)
## Internet VPNs
### Site-to-Site VPNs
VPN used to connect two sites together for the same organization. In this kind of VPN, a 'tunnel' is created that carries data securely from one site to the other. This is done through a protocol called IPsec. The protocol encrypts the original packet and encapsulates it with a VPN header and encapsulates it again with an IP header that will carry it to the other site's edge router.

The devices that are responsible for maintaining this tunnel aren't the hosts themselves but are edge devices; an edge device could be a VPN concentrator (old and stinky), a router, a firewall, etc... These edge devices create and maintain this tunnel so traffic is carried between them but once the encrypted packets arrive at the other edge device, that edge devices decrypt and decapsulate the packets thus ending up with normal IP packets that are then routed to their destinations at the other site as if it were all just one big LAN.

Limitations of IPsec:
- IPsec does not support broadcast or multicast traffic; only unicast traffic. This means routing protocols like OSPF do not work over VPN. To solve this, something like *GRE over IPsec* can be used.
- Configuring a full-mesh of tunnels between many sites is labor-intensive. This can be solved by Cisco's *DMVPN* 
#### GRE Over IPsec
Generic Routing Encapsulation or GRE can create tunnels like IPsec that can be used to connect LANs and supports a wide variety of protocols as well as multicast and broadcast addresses. However it is not secure as it does not encrypt the traffic like IPsec. Thus, you can use GRE to encapsulate the packet first then apply IPsec over it.
#### DMVPN
Dynamic Multi-point VPN is a Cisco protocol that can be used to cut down on labor time. For DMVPN to work:
1. Connect all the sites routers to one main router (hub and spoke topology)
2. Create IPsec tunnels for those connections
3. The hub router, through DMVPN, provides configuration information to all spoke routers pertaining to the creation of more IPsec tunnels to other spoke routers.
4. Spoke routers create IPsec tunnels with all other spoke routers. 
### Remote-Access VPN
Unlike a site-to-site VPN where a host needs to connect to one of either sites to access the other through the VPN concentrator or similar, a remote-access VPN allows individual hosts to connect to a remote site through the internet regardless of where they are. To do this, client software is installed on the users' devices that forms secure tunnels using **TLS** with the remote site's router/firewall/VPN concentrator that acts as a TLS server. After this connection is established, the connected host is *logically* within the LAN of the site they are connected to. This kind of VPN is sometimes also known as *VPN over TLS*

**Fun fact:** the Cisco AnyConnect client used by NYU is an example of remote-access VPN.