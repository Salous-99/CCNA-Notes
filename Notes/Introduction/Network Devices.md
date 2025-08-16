A computer network allows for communication and the sharing of resources between nodes. Here are some kinds of nodes:
![[Pasted image 20250623125908.png]]

A client (shown on the right in the above diagram) is defined as a device that accesses a service offered by a server. This would mean that a server is a device that offers functions or services to clients.

Since these definitions rely only on services rendered, it tells us that a server and client are not tied to specific hardware requirements, a server can also be considered a client with respect to some other server. For example, a server relying on OCSP acts as a client with respect to the OCSP server. Similarly, in a P2P architecture all clients behave as servers as data is requested from them, they also act as clients as they receive said data.

Switches:
- Have many network interfaces for end hosts to connect to (usually 24+)
- Provide connectivity to hosts within the same LAN
- Do **NOT** provide internet connectivity or connectivity between different LANs, that is a router's job.

Routers:
- Have relatively few network interfaces, 
- Provide connectivity between LANs
- Provide internet connectivity

Network Firewalls:
- Protect end host inside
- Can be placed outside or inside your network (before or after the router)
- Monitor and control network traffic based on configured firewall rules
- Next generation firewalls include more advanced filtering capabilities 

There are also host-based firewalls which are applications on your computer that can filter ingress and egress traffic to your PC.

