FTP and TFTP are industry standard protocols used for transferring files over a network. In the context of network engineering, FTP or TFTP may come in handy for updating the IOS version on the network devices; use FTP or TFTP to transfer the new image to the devices' flash memory, install the new image and reboot the device. The steps of doing that would look something like:
1. Download the new image from Cisco's website to your PC (you being the network engineer)
2. Transfer the IOS image to an FTP server that is reachable by the device you are updating
3. Access the CLI of the device and transfer the image to the device using FTP
4. Update and reboot the device.
## TFTP
TFTP is simple(r) (hence the name: *Trivial* FTP), so we'll start with it first. TFTP came after FTP but it isn't a replacement for it; it is a simpler protocol that can be used in a controlled environment for the purposes of quickly and easily transferring files over a network. TFTP only supports two features: transfer a file from server to client, transfer a file from client to server. that is it. It also does not support encryption and does not use a username nor password. TFTP servers listen for incoming connections on UDP/69*, even though UDP is used, TFTP has built-in features within the protocol itself to ensure reliable data transfer.

**\*Note:** even though the server listens for connections on UDP/69, the data transferred will not go through UDP/69, instead another ephemeral port will be chosen as the server's source port and the rest of the communication will happen from that port number. The reason as to why this happens is to allow the server to process many requests simultaneously as opposed to doing them one at a time. The port number used in the communication is the TID or *Transfer Identifier*. FTP works in a similar way.

Reliability is ensured via lock-step communication; for example, the client might start with a read request in order to copy a file from the server and the server will respond with the first data segment, and all data segments must be acknowledged by the receiver before the next segment is sent. If an acknowledgement is not sent, the subsequent data segment is also not sent, when a certain timeout period is reached, an acknowledgement will be re-transmitted. If the roles were reversed and the client was copying a file to the server, the server will be the one that sends the acknowledgement messages but nothing else would change about the dynamic.

Thus, the TFTP process has three phases:
1. Connection: initial request sent by TFTP client to TFTP server, the server responds with the initial data message.
2. Data transfer: client and server exchange data and acknowledgement messages until the entire file is copied.
3. Connection termination: last data message has been sent and received, and a final acknowledgement is sent to the server.
## FTP
FTP uses ports TCP/20 and TCP/21. It supports usernames and passwords for authentication however, the traffic being sent is still not encrypted meaning its not very secure. There is a version of FTP which runs over SSL/TLS that is called FTP Secure or FTPS. There is also a different protocol all together called SFTP which is SSH FTP; this protocol can be used for even greater security.

FTP is more complex that TFTP in that it allows for more operations than TFTP such as listing directory contents, adding/removing directory, renaming files, etc...

Initially, the FTP client will establish a connection to the FTP server on TCP/21. The is the FTP control line and is used to send FTP commands to the server. The actual data transfer will occur over TCP/20 in one of two modes:
- Active mode: server establishes a connection from source port TCP/20 to the client and start the data transfer.
- Passive mode: client establishes a connection to destination port TCP/20 on the server and start the data transfer. This is more common if the client is behind a firewall; firewalls will block connections initiated from outside the network.
## IOS File System
There are multiple kinds of file systems on Cisco devices, you can view them using **show file systems** some are:
- Disk: Storage devices such as flash memory. The IOS is stored here. When the device starts up, the IOS files are transferred to RAM
- Opaque: Used for specific internal functions
- NVRAM: non-volatile RAM, used to store the start-up configuration file.
- Network: external file systems, for example: FTP and TFTP servers.
## Updating the IOS Image
The first step is to transfer the new image to the device's storage. You can view the contents of the flash memory using **show flash**. There you will find the current IOS image file (.bin file). To transfer a new one to the device, we will make two assumptions:
1. The new IOS image is on a TFTP server.
2. This TFTP server is reachable by the device.
### Using TFTP
To copy a file, you can issue the below command from privileged exec mode:
	**copy <source\> <destination\>**
This can be used for any source and destination storage drives (for example from one disk to another) but for TFTP specifically, and specifically for updating the IOS the command will look like:
	**copy tftp: flash:**
After you select TFTP you will be prompted for the IP address of the server, and then for the name of the file you want to transfer (you need to know this beforehand as TFTP does not allow you to list directories), finally you will be prompted for the new filename, this is an optional field and the same name will be chosen by default.

Once you have the image on the device, you can go to global configuration mode to issue the command:
	**boot system flash:<name of new IOS image file\>**
This will force the device to boot using the new image instead of the first image it finds in flash (that is what happens by default).

After this step, exit out of global configuration mode to privileged exec mode and issue the **write** command to save the configurations and also the **reload** command to reboot the device. After rebooting, you can issue the command **show version** to confirm that the new image is being used, and you can delete the old image file using the command:
	**delete flash:<name of old IOS image file\>**
You will be asked to confirm that you want to delete, you can press enter again.
### Using FTP
FTP authenticates using a username and password, so they will have to be configured first using:
	**ip ftp username <username\>**
	**ip ftp password <password\>**
from global configuration mode. 

After that, the steps are almost identical but instead of *tftp*, its *ftp*):
	**copy ftp: flash:** (answer prompts for IP address and filenames)
	**boot system flash:<IOS image\>**
	**exit**
	**write**
	**reload**