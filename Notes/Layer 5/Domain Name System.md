Domain Name System or DNS is used to resolve human readable text (like google.com) to an IP address that can be used over the internet to communicate with a web server. If you want to visit google.com, you first use DNS to figure out what IP address is associated with google.com then you browser crafts and sends an HTTP(S) request to the IP address resolved by DNS.

DNS requests are sent to DNS servers, these are typically built into your plug-and-play router which your computer uses as a local DNS server. A DNS server's IP address is configured either manually or via the DHCP which stands for Dynamic Host Configuration Protocol.

DNS servers will be respond to a request with a DNS record. An *A-record* stores the IPv4 address of the host whilst an *AAAA-record* has the IPv6 address. DNS uses UDP/53 for communication but can also rely on TCP/53 in case the message is larger than 512 bytes in size.

DNS servers as well as the local host itself will cache DNS records so that they do not do the same lookup again if they need the same host. To view the DNS records currently cached:
	**ipconfig /displaydns** on windows

Before DNS, a "hosts" file was used where host names and their IP addresses were stored in key-pair values and kept in this host file for lookup when an IP address was needed. Now, networks have a local DNS server that serves a local network, queries other DNS servers, caches responses and more.

If relying on an external DNS server, there is no need for a local one and your edge router only has to forward traffic between the external DNS server and internal hosts. In this capacity, the router does not take on any additional roles. In order to turn it into a DNS server, you must issue the command:
	**ip dns server**
From global config mode. This enables the DNS process. Then you have to add DNS records with the host names and addresses of the devices using:
	**ip host <host name\> <IP address\>**
You can then issue the command
	**ip name-server <IP of external DNS server\>**
This external server can serve as a backup in case a requested domain name is not within the records. Finally issue the command:
	**ip domain lookup**
This command allows the device to actually craft and send DNS requests (needed for communication with 8.8.8.8)

This makes the router function as a DNS client and server at different times. Functioning as a server when the answers are known and as a client when they're not.

To display the different records cached on the computer, run the command:
	**show hosts**command which allows users
You will notice that manually entered records are permanent whilst DNS records are temporary and will expire and need renewal (sending and receiving a request again)

To configure a router to act as a client only, you need to only issue the commands:
	**ip name-server <IP of external DNS server\>**
	**ip domain lookup**

Finally, there is also the command:
	**ip domain name <name\>** 

which allows a router to append the domain name provided above to any host name without a domain. So if the router receives a DNS request for a pc1 and the configured domain is ccna.exam.org, the actual DNS request it processes is for pc1.ccna.exam.org.

