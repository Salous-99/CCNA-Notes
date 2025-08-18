ACLs have multiple uses, as the name suggests, they control access to resource. They mediate subjects' (users or processes) access to objects (namely data, but could be other privileged processes). ACLs thus function as a packet filter, dropping or forwarding a packet in accordance with a set of rules. ACLs can filter based on:
- source/destination IP addresses
- source/destination port numbers

ACLs are configured on the router itself; consider the following network:
![[Pasted image 20250816111913.png]]
This network will be used to demonstrate how ACLs are used. ACLs should not be configured randomly but they should be configured in accordance with a set of rules, for example, let us say we have the following security requirements:
- Hosts in 192.168.1.0/24 can access the 10.0.1.0/24 network
- Hosts in 192.168.2.0/24 cannot access the 10.0.1.0/24 network

ACLs are an ordered list of Access Control Entries or ACEs. These ACEs are configured within the router's global configuration mode, so our ACL could be:
1. If source IP = 192.168.1.0/24, forward.
2. If source IP = 192.168.2.0/24, drop.
3. Else, forward.

After configuring the ACL, it must then be applied to an interface to work. ACLs are applied either inbound or outbound on an interface. If the ACL were to be applied to outbound traffic on the G 0/2 interface, that would mean it will only apply the filters to traffic going to the 192.168.2.0/24 subnet (which coincidentally would make rule 2 useless). Applying the ACL to this interface would not meet the above requirements. As traffic incoming from .2.0/24 headed to either server will be allowed as the rules are only applied to outbound traffic so traffic coming from .2.0/24 will not be checked against rule 2 and will be forwarded to the servers if that were to be the destination. If the same rules were to be applied but for inbound traffic, it will still not meet the requirements as it will not cause all traffic incoming from .2.0/24 to be dropped, even if it was not meant for 10.0.1.0/24, meaning it is excessively and unnecessarily dropping packets. The best interface to apply the ACL is outbound on G 0/1 of R2. 

The ACEs are evaluated in order, and the first rule to be met is the one that is applied, and the rest of the ACEs are ignored. if none are met, the packet is dropped by default. This is known as ***implicit deny***.

Only two ACLs can be applied on an interface, one in either direction. If you set two different ACLs in the same direction on the same interface, the second ACL will overwrite the first one.
# ACL Types
## Standard ACLs
Standard ACLs filter traffic based on the source IP address of a packet. 
### Standard Numbered ACLs
ACLs identified with a number, e.g. ACL 1, ACL 2, etc... There is a specific range for the ACL numbers for standard ACLs, this range is: 1-99, and 1300-1999, standard ACL numbers must fall into one of these two ranges.

To configure a standard numbered ACL, you can issue the following command in global configuration mode:
	**access-list <ACL number\> {deny | permit} <source IP address\> <wildcard-mask\>**
This command will add one ACE to the ACL with the corresponding number, creating a new ACL if this was the first ACE to be assigned.

You can also add a "remark" to an ACL which is similar to an interface description; it tells you the purpose of the ACL. To do so:
	**access-list <ACL number\> remark <description/remark\>**
In order to apply an ACL to an interface, navigate to that interface's configuration mode and issue the command:
	**ip access-group <ACL number\> {in | out}**
### Standard Named ACLs
These can be identified with a name and configured from a standard ACL configuration mode. First, issue the command:
	**ip access-list standard <ACL name\>** OR
	**ip access-list standard <ACL number\>**
From global configuration mode. You will then enter standard ACL configuration mode. From there, you can issue the command:
	**\[entry number] {deny | permit} <Source IP address\> <wildcard-mask\>**
If the entry number is not specified, the first entry will be numbered 10, and each subsequent entry will increment the number by 10.

To apply the ACL to an interface, it is almost the same command:
	**ip access-group <ACL name/number\> {in | out}**
## Extended ACLs
Extended ACLs work the same as standard ACLs but they have more filtering options. These include:
- Source and destination IP addresses
- Source and destination port numbers
### Extended Numbered ACLs
The number ranges for these are: 100-199, and 2000-2699. To configure an extended ACE, you can issue the command:
	**access-list <number\> {permit | deny} <protocol\> <source IP\> <Destination IP\>**
Thus adding an ACE to the ACL with the given number (remember the extended ACL number range). You can also rely on ACL configuration mode, see next section. 
## Extended Named ACLs 
The command:
	**ip access-list extended <ACL name/number\>**
ran from global configuration mode allows you to enter the ACL configuration mode which is easier to work with. Again, the number for the extended ACLs must fall within the proper range. From within the ACL configuration mode, you can run the command:
	**\[ACE number] {permit | deny} <protocol\> <Source IP\> <Destination IP\>**
## Extended ACL Filtering
### Protocol Filtering
You can use extended ACLs to drop packets that use a specific protocol (defined in the protocol field or next header field of an IP header). For example, you can block all ICMP traffic using **deny icmp**.
### Application level filtering
You can leverage extended ACLs capability of filtering based on UDP and TCP port numbers to control what application data travels through the network. When using protocol filtering to mediate access for UDP and TCP segments, you can define the permitted/denied port numbers to pick and choose specific application level protocols, for example, you can block all HTTP requests (because HTTPS is better) by making the following ACE on your default gateway's uplink interface (outbound direction):
	**deny tcp any eq 80**
This entry means the following: Deny TCP traffic with any source IP address that is communicating on TCP port 80 (port to which HTTP requests are sent on a web server).

For this filtering, you can employ many comparison operators, these are:
- eq X: equal to X
- neq X: Not equal to X
- gt X: greater than X
- lt X: less than X
- range X Y:  from port X to port Y.
Luckily, the Cisco IOS helps you out a little by allowing you to enter the protocol name instead of the port number it uses. This makes for less memorization; i.e. the command above can be rewritten as: 
	**deny tcp any eq www**
This however, does not account for the possibility that a server somewhere is using a well known protocol like HTTP but listening for requests on a port other than 80.

The general format of this command is:
	**\[ACE number] {permit | permit} <protocol\> <Source IP\> <wildcard mask\> \[comparison operator <Source port number(s)\>] <Destination IP\> <wildcard mask\> \[comparison operator <Destination port number(s)\>]**
If the IP address belongs to a host (32 bit mask): **<IP address\> 0.0.0.0** can be replaced with **host <IP address\>**

Because of this ability to track both source and destination addresses, it is best (unlike in standard ACLs) to place the ACL closest to the source. 
## Editing ACLs
In case you incorrectly configured your ACL, you cannot remove entries from global configuration mode using the keyword **"no"**. If you try to remove an ACL entry in this way from global configuration mode, e.g.
	**no access-list 1 permit any**
Instead of removing that one entry, you will remove the whole of ACL 1. In order to have fine-grained control over the ACL, ACL configuration mode is best. To enter this mode, you can issue the command: **ip access-list standard <ACL name/number\>** (also shown above). From there you can display the ACL and then remove any specific entry by issuing the command (from ACL configuration mode):
	**no <ACE number\>**
This will remove that ACE specifically. You can also insert ACEs between other pre-configured ACEs by specifying a sequence number that falls with the range.

If multiple ACEs were configured with sequence numbers that are incremented by 1 only, then you would have ACE 1, ACE 2, ACE 3,... This makes it impossible to insert other ACEs in between as there is no room in the sequence number (I assume this is why sequence numbers go up by values of 10 by default). In order to fix this issue, the ACEs can be re sequenced using the following command ran from global configuration mode:
	**ip access-lit resequence <ACL number\> <Number of first ACE\> <Increment size\>**
# Cisco IOs
## Show commands
To display all access-lists (of all types):
	**show access-lists**
To show IP based access lists only:
	**show ip access-lists**
To display remark, you could look into the running configuration using something like this:
	**show running-config | include access-list**
The pipe followed by the **include** command works similarly to grep. You can also streamline the process further by using the command:
	**show running-config | section access-list**
This will show you the ACL section of the running configuration.

