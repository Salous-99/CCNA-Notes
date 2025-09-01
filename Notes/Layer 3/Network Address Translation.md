*I am unsure if layer 3 is where this one belongs but I can't decide so whatever.*

NAT or Network Address Translation was implemented as a short term solution for IPv4 address exhaustion. Other short term solutions include: CIDR and Private IPv4 addresses. A long terms solution would be to use IPv6.
## Private IPv4 Addresses
The following networks contain IPv4 Private Addresses (specified by RFC 1918):
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16
Private IP addresses are also not globally unique, they are only **locally unique** meaning that two different autonomous systems could both have hosts that share the same IP addresses but since these addresses are private IPv4 addresses, they will not cause a conflict.

Given the fact that private IP addresses are not globally unique, a router cannot know the route to a certain node with a private IP address because there are many nodes with that same IP address and as such, private IP addresses are not routable over the internet. If a router on the internet receives a packet with a private IP address as a destination, it will drop that packet. Routers in general can still route to them, but only do so if its an internal router (internal to an AS) and the destination is also reachable by this router.
## NAT
From what we already know, private IP addresses are not routable online and there are also many duplicate private IP addresses all over the world, there are two important questions still left to answer:
- How can there be many duplicate IP addresses without a conflict happening?
- How do private IP address ranges help save-up IP addresses?
- How do local hosts communicate to the internet when their source IP addresses are not routable?

The answer to all of these questions is Network Address Translation. As for the first question, a router configured with NAT will change the destination IP address from the private address assigned to the host locally to a different **public address** that is routable over the internet. This way, there are no duplicate addresses*. 

the router at the edge of the network will change the source IP address from the host IP (private) to the router's publicly facing interface's IP (public IP that **is** routable), and store that mapping in a table, then when traffic is returning to that host, the destination IP address will be switched from that of the router's interface to the host's address by relying on that previously stored mapping. Since the public IP addresses are globally unique, no conflict will occur and no private IP address will be routed over the WAN.

Second question: each AS utilizes private IP address range(s) for all of its internal networking needs and uses NAT to wrap all of those private IP addresses behind a handful (sometimes just one) public IP address that is used for communication with the internet.

What is the mapping mechanism? Simple: It maps the source network address to the destination address.
### Static NAT
The different ways to do NAT refer to the different ways in which the mappings are done from private to public IP address. Static NAT means mapping *Inside Local* addresses to *Inside Global* addresses.

- **Inside Local**: The IP address of the inside host, from the perspective of the local network. These are usually configured on the inside host and are usually private addresses.
- **Inside Global**: The IP address of the inside host, from the perspective of the outside hots. These are IP addresses of the inside hosts after network address translation has been performed, they are usually public addresses.
  
For example, a local LAN might be assigned 192.168.0.0/24 and a host PC within the LAN has the address 192.168.0.168 and it wants to communicate to server 8.8.8.8. Since the destination is outside the local subnet, the packet is sent to the router, and since 192.168.0.0/16 hosts are not routable, NAT is performed and the public IP address 100.0.0.2 is assigned as the inside global address of that host. The source IP of the packet is switches to 100.0.0.2 and the packet is routed over the internet and when the server responds, the router will switch the destination IP address from 100.0.0.2 to 192.168.0.168 and route the packet to the destination LAN.

**Note:** since one-to-one mappings are required, this doesn't really save up any IP addresses and just solves the "non-routable" issue. 

**Note:** Static NAT has limited uses cases since the so-called "inside global" addresses must be owned by your organization in order to use static NAT.

**Note:** Static NAT also allows outside hosts to access inside hosts since the mapping also applies the other way, i.e. borrowing from the previous example; if an outside host wants to communicate with PC 192.168.0.168, from the perspective of those hosts, that PC has an IP address of 100.0.0.2 and if they send traffic to that IP address, static NAT will still work and it will change the destination IP addresses to 192.168.0.168 and the packet will get through (unless there are access control rules against that sort of thing).
## Dynamic NAT
Dynamic NAT does not differ too much from its static variant; instead of assigned a one-to-one mapping, in dynamic NAT, a pool of inside global addresses are defined and an ACL is configured onto the interface to select what specific traffic to filter, meaning:
- Traffic that meets the criteria of the ACL will undergo the NAT pocess:
	- if its outgoing, the source address will be mapped to the **first available** IP address in the NAT pool.
	- If its incoming, the destination address will be set to the hosts inside local address.
- traffic that does not meet the criteria of the ACL does not undergo network address translation but would still be forwarded if its destination is reachable.
### NAT Pool Exhaustion
If the number of hosts that require NAT is greater than the pool size, then *NAT Pool Exhaustion* will occur. If a router receives a packet that needs an inside global address but there aren't any available, the router will drop the packet instead. Hosts wanting to communicate to the outside must then wait until an IP address becomes available which can happen either due to a mapping expiring and being dropped from the table or if someone manually clears the translations table.

Dynamic NAT is better then static NAT at saving IP addresses but its still not nearly good enough because:
- You need to own all inside global IP addresses in your pool in order to actually use them as part of your NAT pool.
- Needing 10+ public IP addresses per organization will still exhaust all IPv4 addresses very quickly.

Dynamic NAT entries are created when the host initiates a connection with an outside host; the mapping is created and stored in the translations table for 24 hours, renewed whenever the same translation is used. Other dynamic, per-connection entries will be stored in the table whenever a new connection is initiated, these entries expire in 1 minute. These timers can be changed but that is beyond the CCNA. 
## Port Address Translation
Port Address Translation, also known as NAT overload or simply PAT translates both the IP and port number (if necessary). PAT works by including the port number in the mapping as well, this allows multiple hosts to have their IP address mapped to a singular public IP address and the router will identify different streams by identifying the source port numbers. 

Suppose you have the following network from before:
![[Pasted image 20250901154704.png]]
 If PC1 and PC2 were to communicate with server 8.8.8.8, they will do so over numbered TCP and UDP ports; the destination ports usually correspond to a well known service but the source ports are randomly chosen by the hosts from a pool of ephemeral ports. If PC1 and PC2 were to both communicate with 8.8.8.8 and PC1 used TCP/54321 whilst PC2 used TCP/54322, the IP address can be the same (lets say 100.0.0.1 like from earlier) but the router will perform PAT in order to create two dynamic entries:
 - For PC1: 192.168.0.167:54321 -> 100.0.0.1:54321
 - For PC2 192.168.0.168:54322 -> 100.0.0.1:54322
If by some chance, PC1 and PC2 both used TCP/54321 or any other source port number, the router will change the port number to something else so the mapping can work and then reverse the operation when the server responds.

This system is much better at saving IP addresses because you can hide all hosts behind a handful, or even just one IP address.
## Cisco IOS Configuration
### Configuring Static NAT
First you must configure the interfaces that will be performing the NAT. (I am assuming NAT will be performed on packets outgoing from the interface). Navigate to the interface configuration mode of the interface connected to the LAN or the *inside* interface and run the command:
	**ip nat inside**
Then navigate to the interface configuration mode of the public interface and run the command:
	**ip nat outside**
Afterwards, from global configuration mode, you can configure each mapping manually using the command:
	**ip nat inside source static <inside local IP address\> <inside global IP address\>**
**Note:** If you try to map multiple inside local addresses to the same global address, the command will be rejected. Only the first mapping will persist.

These mappings can be shown in tabular form using the command:
	**show ip nat translations**
**Note:** this will also show two other columns containing the *outside local* and *outside global* addresses. These are the IP address of the remote/outside host from the perspective of the local network and the perspective of the outside network respectively.

You can clear all mappings you configured using:
	**clear ip nat translation \***
from **privileged exec mode**. This command will also clear any dynamic mappings as well. Dynamic NAT entries are added whenever an inside host communicates with an outside host; the mapping entry would be different since the outside local and global addresses would be defined. These will eventually expire and be removed from the list or manually cleared by the above command.

You can the command:
	**show ip nat statistics**
That displays statistical information about translations that occurred.
### Configuring Dynamic NAT
You must first define which are the "inside" and "outside" interfaces, just like in static NAT. You also need to configure two other things:
- an ACL that identifies the traffic you want to apply NAT to.
- a pool of inside global addresses your router can use for NAT.

To define an ACL, it is the same command as from the [[Access Control Lists]] section;
	**access-list <ACL number\> {deny | permit} <source IP address\> <wildcard-mask\>**
*unsure if extended ACLs can be used for this purpose. I assume yes.*

To define a pool, run the command:
	**ip nat pool <Pool name\> <first IP address\> <last IP address\> <netmask\>**
The netmask is only there to ensure that both the first and last IP addresses fall within the same subnet, if they aren't, the command will be rejected.

Finally, you can map the ACL to the pool, you can configure dynamic NAT. You can do this by running the command:
	**ip nat inside source list <ACL name or number\> pool <Pool number\>**
### Configuring PAT
Almost identical to to Dynamic NAT in terms of configuration, with just one exception, when applying the ACL to the pool, ensure that you run the command:
	**ip nat inside source list <ACL name or number\> pool <Pool number\> *overload***
The *overload* keyword activates PAT instead of normal dynamic NAT.

Yes you still define a pool of addresses, it can be a lot smaller however, or it could be just one IP address. In fact, you can configure a router to use the public interface's IP address as the sole address used in PAT. To do this, you follow the same steps as before but you do not have to identify a pool of IP addresses. Instead, you can issue the command
	**ip nat inside source list <ACL name or number\> interface <interface\> overload**
This will use the IP address that has been configured on the interface instead of an IP address from a pool.