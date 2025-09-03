Quality of service or QoS is used to prioritize certain traffic flows over others for various reasons. One important and common reason (in the more civilized world) is prioritization of VoIP traffic which is used for IP Phones (phones that communicate over a packet switched network instead of a circuit switched network) so that the audio quality is acceptable.
# IP phones
Older phones worked over the *Public Switched Telephone Network* or PSTN sometimes also called a POTS network which stands for *Plain Old Telephone Service*. IP phones connect to a switch and operate on a traditional IP network.

Cisco IP phones (like the desk telephones in NYUAD) have an internal 3 port switch, 2 externally available; one is connected to the access-layer switch and another is connected to the PC. The third port is not visible for users and is used to connect the phone to the switch (so that the IP phone can actually work). This is done on IP phones to save up ports on the switch; instead of having to connect two devices to the switch, you only connect the IP phone, and you can connect the PC on a workstation to the switch of the IP phone see below:
![[Pasted image 20250903110433.png]]

Note that it is recommended to separate data and voice traffic in separate VLANs. It is also recommend that all voice traffic is placed in a specific voice VLAN that can later be prioritized over the data VLANs.
## Power Over Ethernet
These IP phones require two connections, one power connection, and another RJ-45 connection used to connect to the switch. Using PoE or *Power over Ethernet*, Power Sourcing Equipment or PSEs (like a switch) can use an Ethernet cable to deliver power to the IP phone as well as transfer data.

Examples of Powered Devices or PDs:
- IP Phones
- IP Cameras
- Wireless Access Points
The switch acting a the PSE also converts the power from AC power (it gets from mains) to DC power that can be used to power electronics.

Below is a description of the basic operation of PoE and *Power Policing*:
- When a device is connected to a PoE-enabled port, the PSE ends low power signals, monitors the response, and determines how much power the PD needs.
- If the device needs power, the PSE supplies the power to allow the PD to boot.
- The PSE continues to monitor the PD and provide power as needed (not too much or too little).
![[Pasted image 20250903130258.png]]
# Quality of Service
Modern networks are converged networks which means that IP phones, video traffic, as well as something like HTTP(S) data all share the same IP network. If there is enough bandwidth to go around, then everything is fine, but if the network gets congested and different applications start competing for bandwidth. At this point, you would then have to prioritize certain kinds of traffic to ensure smooth operation. Priority traffic is usually associated with real time applications like video and phone calls; reason being if you have 25% data loss on a file download, you simply take 33% longer to download it, but a phone/video call with 25% data loss is illegible. QoS is a set of tools used by network devices to apply different treatment to different packets.

QoS can be used to managed the following characteristics of a network:
- Bandwidth: overall capacity of a link, measured in bits per second
- Delay: one way delay is the time taken for a packet to get to its destination, two way delay is the amount of time needed for a round trip.
- Jitter: variation in one-way delay between packets sent by the same application.
- Loss: percentage of packets that do not reach their destination. Could be caused by network congestion or faulty hardware/wiring.
For video calls, the recommended maximum values for the above metrics are:
1. delay: 150 ms or less
2. Jitter: 30 ms or less
3. Loss: 1% or less
If this criteria is not met, there will be a noticeable reduction in audio quality.
## Classification and Marking
The purpose of QoS is to prioritize certain traffic streams. How are these streams first identified? This can be done in many ways;
- An ACL: if a packet meets the criteria of an ACL, it will be treated in a certain way whilst other traffic is processed as per the default protocol
- Network Based Application Recognition or NBAR: performs deep layer packet inspection (at layer 7) to identify the traffic stream
- In the layer 2 and 3 headers, there exists some flags that can be used to identify higher priority data:
	- PCP and DEI bits in the layer 2 Dot 1 Q tag (if it exists)
	- DSCP bits in the IP header
### Dot 1 Q Tag
If the frame was tagged, within the tag, there exists 3 bits used for *Priority Code Point* or PCP and it is used to indicate the class of service for that VLAN. The 8 values it can take on represent:
0. best effort (default) - no guarantees
1. Background
2. Excellent Effort
3. Critical Applications - Used by IP phones when establishing a call
4. Video
5. Voice - Used by IP phone for actual voice data
6. Internet Work Control
7. Network Control

This method of including class of service information is known as *marking*.

Given that the dot 1 q tag is not always in the header, this limits the applications of this method of tracking, it is ideal for use in IP phone VLANs.
### IP ToS Byte
IP headers have two fields, these are:
- DSCP: Differentiated Services Code Point
	- This used to be called the IPP (IP Precedence), used to be 3 bits in length and the rest of the byte was unused.
	- Now its 6 bits, and the rest of the byte (2 bits) is used for the;
- ECN: Explicit Congestion Notification

The standard for IPP (old ToS info) is the same as that for PCP: 
0. best effort (default) - no guarantees
1. Background
2. Excellent Effort
3. Critical Applications - **IPP** tag used by IP phones when establishing a call
4. Video
5. Voice - **IPP** tag used by IP phones when transmitting actual voice data
6. Internet Work Control
7. Network Control

As for DSCP, it is defined in RFC 2474 in 1998. When IPP was updated to DSCP, so did the marking used; by having generally agreed upon standard markings for different kinds of traffic, QoS design and implementation is simplified. Also, QoS works better between enterprises. Here are some common standard markings: 
- Default Forwarding - best effort
- Expedited Forwarding - low loss/latency/jitter traffic (usually voice)
- Assured Forwarding - A set of 12 standard values
- Class Selector - A set of 8 standard values, provides backward compatibility with IPP
#### Default and Expedited Forwarding
The DSCP for default forwarding is all zeros: 000000
For expedited forwarding: 101110
#### Assured Forwarding
The DSCP values for these are listed according to the following standard:
- bits 0-2: Class (8 classes could be defined, but in practice only classes 1 through 4 are used)
- bits 3-4: drop precedence
- bit 5: always zero
- \*bits 7 and 8: ECN (not a part of DSCP)

The naming convention for theses different classes of service is given in the form of AF**XY**, where X is the class number and Y is the drop precedence; that said, something like:
	010110XX (XX are the two ECN bits, they are set to don't-cares)
is class 2 (0b010 = 2) and drop precedence 3 (0b11 = 3) as such, this DSCP level is called AF23  

As for how these classes work:
- all packets within the same class have the same priority
- within a class, a higher drop precedence means the packet is more likely to be dropped during congestion
#### Class Selector
The standard here is pretty simple, IPP used 3 bits and CS, which is used for backwards compatibility with IPP, also uses the most significant 3 bits of the DSCP, whilst resetting the rest.
0. 000_000 - DF or best effort
1. 001_000
2. 010_000
3. 011_000 - used to establish calls
4. 100_000
5. 101_000 - used for voice data
6. 110_000
7. 111_000
#### RFC 4954
This request for comment defined industry standard markings to use, these are:
- Voice Traffic: EF
- Interactive Video: AF4x
- Streaming Video: AF3x
- High Priority Data: AF2x
- Best Effort: DF
### Trust Boundaries
The trust boundary defines where devices trust or distrust the QoS marking on a received message, if the markings are trusted, they aren't changed and the frame/packet is forwarded as is. If they aren't trusted, they are changed (the DSCP markings) to something else inline with a preconfigured policy on the device. If IP phones are connected to a switch, it is best to move the trust boundary to the IP phones to get advantage of the QoS markings for voice traffic. This can be done by configuring the switchport connected to the IP phone but that is beyond the scope of the CCNA.
## Queuing and Congestion Management
When a router receives many packets that it has to forward out of one interface, the interface might be busy so the router queues the incoming packets in a FIFO buffer and forwards them as their turns come. Packets continue to arrive, and the queue eventually might fill up, at this point, incoming packets will simply be dropped, this is called ***tail drop***. Tail drop can lead to a phenomenon called *TCP Global Synchronization*, this happens when tail drop occurs, calling all hosts that have TCP connections established to decrease the window size as the hosts perceive  the network congestion, they then gradually increase the window size to get to maximum transmission rate. The only issue is: every other host behind the router is "thinking" the same thing; a packet is dropped, all hosts decrease the window size and restrict how much they send leading to a non-congested network, all of the hosts then gradually pick up speed as they notice that the network is not congested, then the congestion comes back again as all hosts transmit at maximum speed at the same time, leading to other dropped packets leading to a global slow down.

To combat this, you can utilize Random Early Detection or RED. RED is a policy whereby if the device's queue fills up beyond a certain threshold, the device will start randomly dropping packets from select TCP flows. By introducing randomness, we avoid the phenomenon of global TCP synchronization as different hosts decrease and increase their window sizes at different points. In standard RED, all traffic streams are treated the same, if we need to prioritize certain kinds of traffic, then Weighted Random Early Detection or WRED is needed. WRED allows dropping packets depending on *traffic class*.

Another important part of QoS is having multiple queues; the router can classify each packet based on its markings (DSCP or PCP if there) and decide which queue to place this packet in, after which, a scheduler needs to determine what packet to get from what queue because the device can only forward one packet at a time.

There can be many scheduling polices:
- Round robin: the first packet is taken from each queue, one at a time.
- Weighted round robin: similar to regular round robin but each queue has a weight associated with it that determines how many packets are sent from each queue at each go-around. Higher weight = more packets sent from this queue.
- Class-Based Weighted Fair Queuing: weighted round robin but it guarantees that each queue will get a percentage of time from the link, even if the link is congested.
- Low Latency Queuing or LLQ: establishes one or more *strict priority* queues that are always chosen if they contain any packets. These queues can be reserved for something like voice data that cannot handle the latency or jitter of any kind of round robin scheduling. It can however cause starvation of other queues.
## Shaping and Policing
As was previously mentioned, starvation can be caused by an LLQ policy, *policing* prevents that.
### Traffic Shaping
Traffic shaping means reducing the effective transmission rate if it exceeds a certain threshold, even if the interface can accommodate. The packets that would be outgoing are instead stored in a queue and transmitted later.
### Traffic Policing
Traffic policing is a more "strict" version of traffic shaping; instead of buffering packets that exceed the configured transmission rate, the packets are dropped instead.

Policing does still allow for short bursts of traffic to exceed the limit. This can be useful for "burst-y" applications that tend to only communicate in short bursts.
### Usage
A typical and normal and common usage of policing and shaping is your relationship with your ISP; the ISP router that is receiving your data and routing it to the broader internet could use traffic policing to ensure that the bandwidth you actually get is the bandwidth you paid for. Your home router responds by applying traffic shaping to the packets it sends out to ensure that the packets are not dropped by the ISP router.
# Cisco IOS Configuration
## Configuring a Voice VLAN
Consider the following network:
![[Pasted image 20250903110908.png]]
Regular data traffic from PC1 should be on VLAN 10, and voice data coming from the IP phone should be on a separate voice VLAN 11. To configure the port G 0/0, navigate to its interface configuration mode and specify it as an access port using:
	**switchport mode access**

Then set it to accept two VLANs, the first one being the data VLAN, the second being the voice VLAN:
	**switchport access VLAN 10** for the traffic VLAN
	**switchport *voice* VLAN 11** for the traffic VLAN

Even though port G 0/0 carries traffic from multiple VLANs, it is **not a trunk**. It is an access port for two kinds of VLANs. Switch SW1 will utilize CDP to figure out that the connected node is an IP phone and then instruct it to tag VLAN 11 packets that are outgoing from the IP phone whilst leaving traffic data from PC1 without a tag.

PoE configuration is not needed for the CCNA, but here is how the *Power Policing* feature can be enabled; from interface configuration mode for the port connected to a PD, issue the command:
	**power inline police** which is the same command as
	**power inline police action err-disable**
This setting, if configured, would cause the port to be disabled and an error Syslog message to be sent in case the PD draws too much power. To re-enable the port, it must be restarted with
	**shut** followed by
	**no shut**
If you do not want the port shut down but restarted automatically (and an error message sent to the Syslog server), you can use:
	**power inline police action log**