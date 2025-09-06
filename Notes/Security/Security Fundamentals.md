The CCNA is not a Cybersecurity cert, nonetheless, network associates need to know the basics of security in order to work effectively.
# Key Security Concepts
What is the purpose of security in an enterprise? Enterprises should maintain these axioms:
- Confidentiality: only authorized subjects get access to the objects
- Integrity: Data should not be tampered with by unauthorized users
- Availability: The resources should always be operational and accessible to authorized users
Any attack on a computer system aims to target one of these three things. Here is some terminology:
- Vulnerability: weakness that can be compromised to thwart one of the elements of the CIA triad.
- Exploit: a weaponized vulnerability, i.e. a vulnerability that was exploited by an attacker successfully.
- Threat: Potential of a vulnerability being exploited
- Mitigation Techniques: something that can be used to mitigate or protect against a threat
# Common Attacks
## Denial of Service
Attack on availability of a resource, usually a web server; the steps of this attack are:
- The attacker computer(s)* initiate a connection with a web server but does not complete the connection
	- This involves the initial three way TCP handshake
	- The attacker computer sends a SYN, the server responds with a SYN-ACK
	- the attacker computer does **not** respond with the final ACK message.
- The attacker computer(s) do this multiple times, flooding the servers with incomplete TCP handshakes
- Eventually, the target computer's (server's) TCP connection table fills up and it can no longer initiate further TCP connections.
- After a certain time of inactivity the server will drop the connection itself, but it won't matter because the attacker computer(s) have already sent 500 more requests in the time it took the server to timeout one connection.

	**\*Note:** It is more common nowadays for one attacker computer to control hundreds or thousands of "zombie computers" that form a *bot-net* that act on the attacker's behalf, so most of the time, the source IP address isn't even the attacker, it is likely to not even be the source IP address of the zombie, but instead a spoofed address. This attack is called a ***Distributed* Denial of Service** attack or DDoS as it is most commonly known.  
## Spoofing Attacks
### DHCP Exhaustion Attack
The attacker computer sends a DHCP discover request to the local network with a spoofed MAC address, causing the local DHCP server to reserve an IP address within the local subnet, the attacker computer then sends another DHCP discover request but with a different spoofed MAC address and reserves another IP address, it continuously does this until there are no more IP addresses to be had, thus preventing other legitimate users from getting an IP address, thus preventing them from connecting. This makes a DHCP Exhaustion Attack an attack on availability.
### Reflection/Amplification Attack
A reflection attack is one in which the attacker computer spoofs its own IP address to be that of the target, afterwards, it sends traffic to a **reflector** which responds to the incoming traffic but instead of sending the data to the attacker that initiated the requests, it sends traffic to the target's computer since the source IP was spoofed to be that of the target's.

A reflection attack becomes an amplification attack if the response from the server is much larger than the initial traffic sent by the attacker. An example of this would be to do an amplification attack using DNS as a DNS request is relatively short, compared to the response.

Regardless, as the target receives more and more traffic that was not intended for it, its buffer fills up and it is no longer able to effectively process incoming traffic.
### ARP Spoofing Attack
An attacker within the same local network as the victim will wait until the victim tries to ARP for some IP address, after which, the attacker will wait until the legitimate response is sent (it can't know for sure but it can define heuristics to establish when this has likely happened) and then sends its own spoofed ARP response that tells the attacker that the MAC address of the IP address its looking for is the MAC address of the attacker's computer. What this means in practice is that when the target communicates with the IP address it ARP'd for, it is actually communicating directly with the attacker that can then decapsulate and inspect the packet's contents, afterwards, the attacker can simply forward the traffic to the actual destination so that the target isn't made aware of the attack. Since the attacker can read and potentially even modify the contents of the intercepted packets, this is an attack on confidentiality and integrity.
## Reconnaissance Attacks
These aren't attacks necessarily as the information gathering being done relies on publicly available information, the attacker can learn the IP address of a server, what ports are open on that server, physical addresses, phone numbers, etc... by simply googling stuff.
## Malware
Malware means malicious software, there are multiple kinds of malware, all of which differ, not in what they do to the target computer, but in how they spread and infect target systems.
### Viruses
Infect other software on the host computer. Typically spread via user interaction (user shared it, clicked a malicious link, downloaded software from a shady website, etc...)
### Worms
Do not need user interactions to spread as they are pieces of standalone malware that can self propagate across a network, possibly causing congestion.
### Trojan Horses
Harmful software that is disguised as normal software. Can infect a system via malicious links, downloading suspicious content, etc...
### Payloads
The payload carried by a piece of malware can be almost anything, it largely depends on what the attacker wants; if they want money they can create a worm that spreads across hosts in a network and then encrypt all the files on those hosts and hold them for ransom.
## Social Engineering
Competitive lying lmao. There are types of social engineering
### Phishing
Involves sending fraudulent emails pretending to be someone you are not. Has two main sub-types:
- Spear phishing: targeted phishing attack
- whaling: also targeted phishing attack but the target is a big shot.
### Vishing
Voice phishing, same thing as phishing but over the phone
### Watering Hole Attacks
Involves attacking a server or resource frequently used and trusted by the victims, if the trusted website is loaded up with some malicious links and other similar things, the victims will give those links less scrutiny than they should.
### Tailgating Attack
Following someone as they go to a restricted area with the hopes they will hold the door open for you.
# Passwords and MFA
### Guessing
unlikely but mathematically possible
### Dictionary Attack
guessing but advanced. Instead of guessing random things, you guess only real words and permutations of those real words.
### Brute Force Attack
Guessing but dumber. Shouldn't really work...ever...mathematically. 
## Multifactor Authentication
Providing two or more factors of authentication. For MFA to work really effectively, the two or more factors have to come from different categories:
- Something you know: like a password or the answer to a personal question
- Something you have: key fob, phone, badge, RFID tag, etc..
- Something you are: retina, fingerprint, voice, facial features
## Digital Certificates
Used by websites to prove that they are the websites they are stating they are.
# AAA
Framework for monitoring computer systems, the AAA stands for:
- Authentication: verifying a user identity
- Authorization: granting the level of access afforded to that user.
- Accounting: recording user activities on a system 
Large enterprises have AAA servers that manage that automatically, examples of these are:
- RADIUS: open standard protocol, uses UDP/1812 and UDP/1813
- TACACS+: Cisco proprietary protocol, uses TCP/49
	- Cisco has an ISE server which stands for Identity Service Engine, this is the AAA server for Cisco
# Security Program Elements
What can be done to improve overall security?
## User Awareness
Teach users about the basics of cybersecurity hygiene, send out fake phishing emails to teach users how to spot one when they see a real one. Offer dedicated training sessions that teach users about security policies of the company as well as other cybersecurity best practices.
## Physical Access Control
Limit physical access to the resource within the company only to those who need it; someone in accounting does not need server access, so make sure he can't get in. There are multifactor locks that can help introduce MFA into physical security.