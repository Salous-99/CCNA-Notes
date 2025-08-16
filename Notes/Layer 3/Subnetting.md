The time is finally upon us. Today we learn why everyone is freaking the fuck out about subnetting.

When describing networks, we use CIDR notation (read as "cider" as in apple cider). What CIDR stands for is Classless Inter-Domain Routing. Why classless? Because it doesn't really care about the IPv4 address classes, see a list of them in [[IPv4 Addressing]]. 

Why were classless networks needed? Because networks classes lead to many IP addresses being wasted. The restrictions on networks as per the class system was:
- Class A networks have 8 network bits and 24 host bits
- Class B networks have 16 network bits and 16 host bits
- Class C networks have 24 network bits and 08 host bits

This means that if your organization has about 5000 end hosts, a class C network would be too small (hosts up to 254 devices only) whereas a class B network would be far too large; hosting up to 65534 devices, meaning your organization would needlessly take waste 60000 IP addresses. IP address exhaustion thus became a real problem, and many solutions were introduced. Who regulates all of this? whose "problem" is it? That would be the IANA or the Internet Assigned Numbers Authority, it is their job to assign IP addresses to people who need them, but the people who created the internet did not think it would grow this large and didn't think it would be a problem.

How was this resolved? The restrictions above were removed, now even the larger networks can be broken down into sub-networks or subnets.

See below, another example of IP address waste in "point-to-point" networks:
![[Pasted image 20250706144436.png]]
The portion connecting the two routers is a network in and of itself with many many wasted IP addresses under the normal system;
- A class C network can have 254 hosts (256 - 1 network and broadcast addresses)
- One IP assigned to R1's interface
- Another IP assigned to R2's interface
- 252 addresses wasted.

If this class C network were to be broken down further however, you can reduce the number of wasted addresses tremendously. Let's say instead of the 24 network bits typically used for a class C network we use 30 network bits instead which would mean:
- 32 (total address bits) - 30 (network bits) = 2 (host bits)
- 2 host bits can accommodate 2^2 - 2 = 2 hosts.
	- .0 used for the network address
	- .3 used for the broadcast
	- .1 and .2 used for the two routers
This way there are no wasted addresses. Of course in a point to point network such as the one shown above, the actual netmask is /31. Even though a 31 network bits leave no usable addresses (2^1 - 2 = 0), an exception can be made specifically for point-to-point networks since they only have the two addresses. R1 would be assigned 203.0.113.0/31 and R2 would be assigned 203.0.113.1/31.

![[Pasted image 20250706150753.png]]
Above you can see the different subnet masks for subnets of varying sizes.

The CIDR notation allows us to use whatever prefix length we want to break down a large network into as many pieces as we want. There are two special cases to this:
- /31 mask is reserved for point-to-point networks (see above)
- /32 mask is used to specify a host directly; for example if you were defining a route to a specific host on a router, you would use a 32 bit mask.

This kind of subnetting uses fixed length subnet masks or FLSM. There is also VLSM where the V stands for variable. VLSM allows you to use your network more efficiently. Look at the following diagram:
![[Pasted image 20250706224200.png]]
If you were assigned a class C network with address 192.168.1.0/24, you can split it into multiple varying size subnets by following these simple rules:
1. Assign largest subnet at the start of the address space
2. Assign second largest subnet after it
3. Repeat until all subnets have been assigned.