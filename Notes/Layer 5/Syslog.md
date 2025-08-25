Syslog is an industry standard protocol for message logging. It is used to report events occurring within a network in a specialized format. The messages are displayed to the CLI (if it's in operation), stored in the devices' RAM or logged in an external Syslog server. Syslog works in conjunction with SNMP for monitoring, troubleshooting and diagnostics.

The format of a Syslog message is:
	seq : timestamp : %facility-severity-MNEMONIC : description
In turn, these are:
- seq: sequence number of the message (for ordering)
- timestamp: time and date the message was generated
- facility: value indicates which process running on the device generated the message
- severity: a number indicating how severe the logged event is
- MNEMONIC: a short code for the message, indicating what happened
- description: detailed explanation of what happened and what is being reported on.

**\*Note:** The sequence number and timestamp may or may not show up depending on the device's configuration. In order to configure a device to show the sequence numbers and time, you would have to issue the following command in global configuration mode:
	**service timestamps log {datetime | uptime}**
	**service sequence-numbers**
The first command accepts "datetime" which would display the date and time when the event occurred whereas the "uptime" option will replace the date and time with a timer of how long the device has been on when the event occurred (easier to parse through I assume)  

The severity levels are:
0. Emergency: system unusable.
1. Alert: Actions must immediately be taken 
2. Critical
3. Error
4. Warning
5. Notice: normal but significant condition (also known as notification)
6. Informational
7. Debugging
^^ These are specific to Cisco as severity levels are subjective and differ from one manufacturer to the next.

Syslog will log messages in these locations depending on the configurations of the device:
- Console lines: Syslog messages will be displayed in the CLI if the console port is connected and someone is using the CLI. Messages of all severity levels (0-7) will be displayed.
- VTY (Virtual Terminal Lines): Syslog messages will also be displayed in a remote terminal opened using SSH or Telnet. This is disabled by default however
- Buffer: Syslog messages will be saved to RAM. You can view the entire backlog by using the command: **show logging**
- External server: Syslog can be configured to send messages to an external server that logs and consolidates information from many devices. This Syslog server will listen for incoming traffic on UDP/514.

**\*Trivia:** VTY stands for **V**irtual T**TY**. TTY stands for "teletype". A teletype is an old-time keyboard. Also looks like a big goofy typewriter. A teletype was used to transmit characters to the terminal of Cisco devices over the console port, a virtual TTY is used to connect remotely/virtually to Cisco devices' terminals.  
## Syslog and SNMP
They both do similar things; they are used for live motioning of the network's status. They are complementary but their functionalities differ;
	**Syslog** is used for message logging:
		> Events that occur within the system are categorized based on facility/severity and logged
		> Used for system management, analysis, and troubleshooting
		> Messages are sent from the device to the server.
		> The server can't pull information or query the devices like SNMP can with the Get and Set functionalities
	**SNMP** is used to retrieve and organize information about the SNMP managed devices:
		> IP addresses, interface status, temperature, CPU usage, etc...
		> SNMP servers can query devices for information and modify the values of variables within the MIB.
## Cisco IOS Configuration
You can increase or decrease the level of severity for which messages are logged. By default, messages of all severity levels are logged, but that can be changed using the following command issued from global configuration mode:
	**logging console <severity level\>** to change logging level for console/CLI
	**logging monitor <severity level\>** to change logging level for VTY lines
	**logging buffered \[size\] <severity level\>** to change logging level for RAM

Some things to note:
- Severity level can be a number (0-7) or word (Emergency-Debugging)
- Messages of that severity level or higher will be logged. e.g. if the passed severity level is 5 or "notice/notification" messages of severity levels 5, 4, 3, 2, 1, and 0 will be logged as well.
- The second to last command sets the logging level however, no messages will actually be displayed to your SSH or Telnet terminal as the option is disabled by default. To enable it, from the remote terminal, issue the command **terminal monitor** from *privileged exec* mode. This command will allow Syslog messages to be displayed on the remote terminal. Furthermore, it must be re-run with every new terminal session.
- The last command has an optional size field for the buffer size in RAM that will store the logged information. It should not be made too large because it will then take more RAM from processes that need it.

To configure an external Syslog server, use the command:
	**logging <Syslog server IP address\>** OR
	**logging host <Syslog server IP address\>** 

Afterwards, if you want to change the logging level for that server, the command is:
	**logging trap <severity level\>**

