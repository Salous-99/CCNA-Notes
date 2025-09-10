# Virtual Servers
Cisco is primarily known for its network devices but it also offers UCSs, UCS stands for Unified Computing System which is a computer that usually sits on the server room on a rack.

Before virtualization, a physical server would run a few, sometimes just one service; you would have a physical server for a mail server, another for a Web server, etc.. whilst inefficient, it is safer to do it this way then run multiple services on one single physical server as doing so is "putting all of your eggs in one basket" so-to-speak; if one service fails or crashes, one service has a vulnerability that can grant an attacker root access, etc... all your services go down, so it was best to separate them at the physical level. The inefficiency is obvious; many physical servers are expensive: monetarily, power consumption-wise, takes up space, requires more cooling, etc... it also is inefficiency in terms of hardware utilization; a server running one thing is not using 100% of its RAM, NIC (Network Interface Card) slots, CPU power, etc... 

With virtualization, we can break out of the one-to-one model by allowing multiple services to run on one physical server but isolate them logically into multiple *containers* this way you can avoid the inefficiency of multiple underutilized physical servers and maintain a relatively low cost.

A virtual server looks like this (logically):![[Pasted image 20250909134046.png]]

Instead of running one OS on bare-metal, you run a native/bare-metal/type 1 hypervisor (examples include: VMware ESXi and Microsoft Hyper-V), create many virtual machines each running its own OS (could be the same OS, depends on the running application though), and on each OS, one or more applications run to provide a service. Because of the logical isolation, one service crashing will not crash the other services.

There are also type 2 hypervisors, sometimes known as *hosted* hypervisors, are run as another application on a guest OS to run virtual machines with other operating systems. This kind of hypervisor does not see common usage in data centers owing to its inefficiency but is used for personal computers.

With regards to connectivity, the VMs can be connected to the network like so:
![[Pasted image 20250909140517.png]]
The hypervisor also instantiates a virtual switch that can connect to all the virtual hosts' virtual interfaces through the virtual NIC. This traffic is then forwarded by the virtual switch (which has access to the server's hardware APIs) to the outside network through the server's physical NIC.

A virtual port channel can be made to connect the two NICs to two switches for redundancy. This is commonly used in data centers.
# Cloud
Traditional IT infrastructure deployment took two forms or a hybrid of those two forms:
- on premises: data centers located within company property, running on servers bought with company money and operated and maintained by company workers.
- Co-location: a third party data centers sell a space that a company can rent out and put their servers, network devices, and other related devices within that space. The data center is then responsible for electricity, cooling, as well as the space obviously, but it would still be the company that purchases the needed equipment and maintains it.
The cloud/cloud computing is offering an alternative that is becoming increasingly popular.
## The Five Essential Characteristics of Cloud Computing
### On-demand self-service
A consumer can unilateral provision computer capability such as server time and network storage as needed automatically without requiring human interaction with the service providers
### Broad network access
Capabilities are available over the network and access through standard mechanisms that promote use by heterogeneous thin and thick client platforms (e.g. mobile phones, tablets, laptops, and workstations). This means that cloud services should be accessible through any kind of device that can connect to the internet, you shouldn't need a special device.
### Resource pooling
The provider's computing resources are pooled to serve multiple consumers using a multi-tenant model, with different physical and virtual resources dynamically assigned and reassigned according to consumer demand. There is a sense of location independence in that the customer generally has no control or knowledge over the exact location of the provided resources but may be able to specify a location at a higher level of abstraction. Examples of resources include storage, processing, memory, and network bandwidth. 
### Rapid elasticity
Capabilities can be elastically provisioned and released, in some cases automatically, to scale rapidly outward and inward commensurate with demand. To the customer, the capabilities available for provisioning often appear to be unlimited and can be appropriated in any quantity at any time.
### Measured service
Cloud systems automatically control and optimize resource use by leveraging a metering capability at some level of abstraction appropriate to the type of service (e.g. storage, processing, bandwidth, and active use accounts). Resource usage can be monitored, controlled, and reported, providing transparency for both the provider and consumer of the utilized service.
## The Three Service Model of Cloud
### Software as a Service SaaS
The capability provided to the consumer is to use the provider's applications running on a cloud infrastructure. The applications are accessible from various client devices through either a thin client interface, such as a web browser, or a program interface. The consumer does not manage or control the underlying cloud infrastructure including network, servers, operating systems, storage, or even individual application capabilities, with the possible exception of limited user-specific application configuration settings.

Examples:
- Microsoft office 365
- Google's G-Suite 
### Platform as a Service PaaS
The capability provided to the consumer is to deploy onto the cloud infrastructure consumer-created or acquired applications created using programming languages, libraries, services, and tools **supported by the provider**. The consumer does not manage or control the underlying cloud infrastructure including network, servers, operating systems, or storage, but has control over the deployed application and possible configuration settings for the application hosting environment.

Examples:
- AWS Lambda
- Google App Engine
### Infrastructure as a Service IaaS
The capability provided to the consumer is to provision processing, storage, networks, and other fundamental computing resources where the consumer is able to deploy and run arbitrary software, which can include operating systems and applications. The consumer does not manage or control the underlying cloud infrastructure but has control over operating systems, storage, and deployed applications; and possibly limited control of select networking components (e.g. host firewalls.)

Examples:
- Amazon EC2
- Google Compute Engine
## The Four Deployment Models of Cloud
### Private Cloud
The cloud infrastructure is provisioned for exclusive use by a single organization comprising multiple consumers. It may be owned, managed, and operated by the organization, a third party, or some combination of them, and it may exist on or off premises.
### Community Cloud
The cloud infrastructure is provisioned for exclusive use by a specific community of consumers from organizations that have shared concern. It may be owned, managed, and operated by on or more organizations in the community, a third party, or some combination of them, and it may exist on or off premises. 
### Public Cloud
The cloud infrastructure is provisioned for open use by the general public. It may be owned, managed, and operated y by a business, academic, or government organization, or some combination of them. It exists on the premises of the cloud provider. This is the most common kind of cloud deployment model.
### Hybrid Cloud
The cloud infrastructure is a composition of two or more distinct cloud infrastructures that remain unique entities, but are bound together by a standardized or proprietary technology that enables data and application portability. For example, a private cloud which can offload to a public cloud when necessary.
## Benefits of Cloud Computing
- Cost
- Global scale
- Speed/Agility
- Productivity
- Reliability
## Connecting to the Cloud
![[Pasted image 20250909154940.png]]
- Through an agreement with your ISP
- Directly through the internet (less safe)
- Through the internet but over an IPsec VPN tunnel.
# Containers
Containers are similar to VMs but are subtly different. Containers are software packages that contain one application (usually) and all dependencies for that application, but nothing else; making them lightweight, memory efficient, and easier/quicker to deploy in comparison to a regular VM. Multiple applications can be run inside of a single container however this is not typically done.

Containers run on a *Container Engine*, most commonly Docker. A container engine is to containers what a hypervisor is to virtual machines. The differences between a container and a VM however is that a container does not run a separate operating system as a guest operating system; it runs another instance of the same (or similar) operating system as the host OS. The speed boost comes primarily from this fact. To elaborate further; typically a container engine is run on a linux OS, and all containers deployed onto that engine are typically also Linux, if not the same OS exactly, it would be different variants of a UNIX-based OS. 

A *Container Orchestrator* is a software platform that automates the deployment, management, scaling, and maintenance of containers, the most popular platform by far is Kubernetes, designed by Google. Docker Swarm is Docker's own platform. Container orchestrators are required for container management for larger services that require thousands of containers. These are typically *Micro-services*.

***Micro-service Architecture*** is an approach to software architecture that divides larger solutions into many many smaller parts.
## VMs vs Containers

| VMs                                                                        | Containers                                                                                    |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Can take minutes to boot up                                                | Boots up in milliseconds                                                                      |
| Takes up more disk space, gigabytes' worth                                 | Takes up less disk space, megabytes' worth                                                    |
| Uses more hardware resources                                               | Uses much less hardware resources                                                             |
| Portable and can move between physical systems running the same hypervisor | Even **more** portable and can run on nearly any container service                            |
| More isolated from one another; conducive to security and reliability      | Less isolated as they all run within the same OS; if the OS crashes, all the containers crash |
VMs are still widely used but there is a push for container usage over VMs.
# Virtual Routing and Forwarding (VRF)
VRF is used to divide one router into multiple virtual routers; similar to how VLANs divide a switch. Doing so would allow a router to maintain multiple routing tables that are distinct from one another, and it allows the interfaces to be separated on the basis of which virtual router they belong to thus different routing decisions can be made. Some notes about VRF:
- Interfaces and routes are configured to be in a specific VRF aka *VRF Instance*.
- Traffic in one VRF instance cannot be forwarded out of an interface in another VRF.
	- Interfaces not assigned a VRF are also isolated from interfaces that are assigned to VRF, so if you create two VRFs and assign each of them some interfaces, leaving 2 interfaces unassigned, you are effectively creating three VRFs.  
- VRF is commonly used to facilitate MPLS, however, this is not required knowledge for the CCNA, CCNA only requires VRF-lite which is VRF without MPLS.
- VRF is commonly used by service providers to allow one device to carry traffic from multiple customers;
	- Each customer's traffic is isolated from other customers' traffic
	- Customer IP addresses **can overlap with no issue**

That last part saves up some IP addresses as well as simplify or at least standardize the IP assignment process. **VRF configuration on Cisco IOS is also not required**, nor is it doable on Packet Tracer. 