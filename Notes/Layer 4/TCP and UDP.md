TCP and UDP are transport protocols that are responsible for the transference of data between instances of an application running on different hosts.

TCP is an example of a Reliable Data Transfer Protocol and consequently provides the following services/guarantees:
- error recovery
- data sequencing
- flow control

transport layer protocol addressing relies on *ports*. Certain ports are reserved for a specific service, for example HTTP uses port 80, SSH uses port 22, etc...

For addressing, identifying a session works by combining source and destination port numbers. For example, let us say that one end host wants to access two websites using HTTPS (so two applications running on two different servers, but both using the same designated port number of 443), how does the user's end host differentiate between the two sessions? It relies on a combination of both the destination port number (443) and a random source port number. When the server responds, that unique combination will be the same but the roles switches (source becomes destination and destination becomes source). The designated port numbers for well known services (like HTTPS) are assigned by IANA.

The port ranges are designated as follows:
- 0 - 1023: Well-known port numbers - strictly regulated and controlled
- 1024 - 49151: Registered port numbers - aren't extremely regulated but still used for services
- 49152 - 65535: Ephemeral/dynamic port numbers - used as randomly picked source ports.
## TCP
- Connection oriented: establishes a connection first before starting to send data.
- TCP provides a reliable connection; meaning the receiver has to send acknowledgements, confirming to the sender that it did indeed receive the data
- TCP provides sequencing allowing receiving hosts to order the segment in the correct order if they arrive out of order (which can happen)
- TCP provides flow control; meaning the receiver can inform the sender to increase or decrease the rate of data transmission based on the receiver's capability.

TCP header:
![[Pasted image 20250804150225.png]]

When hosts first establish a connection they go through the process of a *Three Way Handshake*; dubbed so as it involves three messages going back and forth. As part of the header, there are two flags: SYN and ACK that are used as part of the handshake. The three steps of the handshake are:
1. Sender sends a TCP segment with the SYN flag set
2. Receiver receives this flag and responds with a segment where the SYN and ACK flags are set
3. The sender responds with a TCP segment with the ACK flag set.

Terminating a connection is sometimes called a 4-way handshake, using the flags FIN and ACK. The termination happens like so:
1. The sender sends a segment with the FIN flag set
2. Receiver responds with an ACK
3. Receiver sends another segment with the FIN flag set
4. Sender sends an ACK to receiver, closing the connection on both ends.

What do these segments actually contain? It's all in the header and header flags. The reason behind the handshake isn't simply to exchange SYN and ACK flags but it is to agree on a sequence number. Sequence numbers are used to reorder segments that arrived out of order, it is also used as a way of keeping track of what was received. The three way handshake establishes sequence numbers for both sender and receiver to use to keep track of the other's sequence number.

The fist SYN segment sent contains the sender's sequence number of N where N is randomly generated. The receiver replies to this SYN with a SYN, ACK segment which contains its (the receiver's sequence number), and it also acknowledges the sequence number of the sender. The sender then finally acknowledges the sequence number of the receiver, and now both parties know what sequence number the other is expecting.

TCP uses forward acknowledgment which means that the receiver sends an ACK of the N + 1 packet where N is the sequence number. This means that if the receiver receives packet with sequence number 43, it will respond with an acknowledgement for sequence number 44, indicating to the sender that it is ready to receive segment 44.

TCP doesn't doesn't require that every segment is acknowledged (that would be inefficient) instead it utilizes a sliding window where the receiver needs to only send an acknowledgement after a certain number of packets were received and ordered, the window size is adjusted dynamically as per the processing power of the receiver.

## UDP
- Connection-less: doesn't establish a connection before transmitting data
- Doesn't provide sequencing
- Isn't reliable
- Does a best-effort job of sending traffic but does not re transmit data if anything is lost 

UDP header:
![[Pasted image 20250804151944.png]]

## TCP vs UDP
- TCP uses more features then UDP at the cost of additional overhead due to the larger header size
- TCP is used where complete and reliable data transmission is needed (no best-effort, only perfection)
- UDP is faster, making it the better choice for real time applications like VoIP calls or video calls or the like (gaming also)
- UDP does not offer reliability meaning the applications has to take up the burden of handling errors.
- Some applications use both depending on the situation.

### Well-known protocols and their port numbers
TCP:
- 20 - FTP data
- 21 - FTP control
- 22 - SSH
- 23 - Telnet
- 25 - SMTP
- 80 - HTTP
- 110 - POP3
- 443 - HTTPS

UDP:
- 67 - DHCP server 
- 68 - DHCP client
- 69 - TFTP (Trivial FTP)
- 161 - SNMP agent
- 162 - SNMP manager
- 514 - Syslog

Both:
- 53 - DNS

