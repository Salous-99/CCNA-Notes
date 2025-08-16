ACLs have multiple uses, as the name suggests, they control access to resource. They mediate subjects' (users or processes) access to objects (namely data, but could be other privileged processes). ACLs thus function as a packet filter, dropping or forwarding a packet in accordance with a set of rules. ACLs can filter based on:
- source/destination IP addresses
- source/destination port numbers

ACLs are configured on the router itself; consider the following network:
![[Pasted image 20250816111913.png]]
This network will be used to demonstrate how ACLs are used. ACLs should not be configured randomly but they should be configured in accordance with a set of rules, for example, let us say we have the following security requirements:
- Hosts in 192.168.1.0/24 can access the 10.0.1.0/24 network
- Hosts in 192.168.2.0/24 cannot access the 10.0.1.0/24 network
This might be the case