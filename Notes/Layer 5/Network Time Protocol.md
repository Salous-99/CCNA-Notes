All devices have internal clocks. These clocks maintain the current time but will drift over time due to hardware imperfections. Highly accurate clocks are very expensive so including an atomic clock in each router is not ideal.

Why do we need accurate time data? For maintaining accurate logs. The tool **Syslog** relies on accurate time data to consolidate the event reports into a log.
# Manual Clock Configuration
You can issue the commands:
	**show clock** OR
	**show clock detail**
To display information about the time and the clock source. The default clock source is the hardware's internal clock. The default time zone used to display the current time is UTC.

To manually set the time, you can issue the command:
	**clock set <hh:mm:ss> <day 1-31> <month 1-12> <year 1993-2035>**
from privileged exec mode since these settings are not a part of the running configuration of the device. The command also changes the clock source from the default internal clock to "user configuration". The internal hardware clock can be configured separately using the command
	**calendar set <hh:mm:ss> <day 1-31> <month 1-12> <year 1993-2035>**
You can then synchronize the calendar (internal clock) and the clock (software clock) using either:
	**clock update-calendar** which synchronizes the calendar time to the clock time OR
	**clock read-calendar** which synchronizes the clock time to the calendar time

To configure the clock timezone, from global configuration mode, issue the command:
	**clock timezone <timezone name\> <hours offset -23-23\> \[minute offset from UTC 0-59\]**
**\*Note:** Updating the timezone will automatically shift the current time to what it would be in the new timezone, so if you are configuring a new device, it is best to configure the timezone first, then set the clock.

**\*Note 2:** There is no way in hell anyone has to manually do all of this.

Anyway, daylight savings' configuration; from global config mode, you can issue the command:
	**clock summer-time <name\> <date/recurring\> <start: first/last/1-4\> <start day\> <start month\> <start hh:mm\> \[start minute offset 1-1440] <end: first/last/1-4\> <end day\> <end month\> <end start hh:mm\> \[end minute offset 1-1440]**

Very long command, the oprtions are:
- name: name of timezone, doesn't have to be real.
- date/recurring:
	- date: absolute summer time, meaning it will be set to summer time at that date and will continue to be summer time until it is reconfigured.
	- recurring: summer time will start and end on set days. The above large command assumes you pick the recurring option which is far more useful.
	- Two sets of time, the first vector has the start time (where summer time begins) and the end time is where the summer time ends. The format is:
		- first/last/1-4: week within the month
		- day: name of the day (sat->Fri)
		- month: name of month (jan->dec)
		- hh:mm: hour and minute
		- (optional) 1-1440: offset in minutes (set to 60 minutes by default because daylight savings usually changes the clock by 1 hour exactly)
# Network Time Protocol
Network Time Protocol or NTP allows us to do what we did above but at large scale and automatically. This is necessary as doing the above configuration on each device takes a long time. NTP allow devices to synchronize their own time to the time of an authoritative NTP server. An authoritative clock source is more trustworthy than the devices' internal clock.

A device using NTP could either be an NTP client or a server, and it can also be both at the same time. An NTP client sends time requests to the NTP server. NTP allows for accuracy within 1 millisecond if the NTP server were to be in the same LAN as the client, and within 50 milliseconds if the NTP is connected over WAN. The proximity to the NTP server determines the accuracy, this distance to the reference clock is called *stratum*.

NTP uses UDP/123 to communicate, aside from needing to communicate with NTP clients, they also need to communicate with other NTP servers as well as the reference clock devices themselves.

A reference clock is at **stratum 0**. It is considered the most accurate, each hop away from the reference clock is a higher stratum. Stratum 1 servers are also known as primary NTP servers. Secondary NTP servers request the time from primary servers or other secondary NTP servers (stratum 2+). The strata go up to stratum 15 after which, an NTP server is considered unreliable.

NTP servers can communicate with servers on higher strata, even sending requests and receiving the time from multiple NTP servers, and they can also peer with other servers within the same stratum to get more accurate time data, this mode of operation is known as Symmetric active mode. A device can operate in client, server, and symmetric active mode all at the same time as well.
## NTP Configuration
### NTP Client
In global configuration mode, you can configure the device to use an NTP server using the command:
	**ntp server <NTP server IP address\> \[prefer]**
You can (and should) configure multiple NTP servers for redundancy and accuracy. You can also favor a particular server by adding the keyword "prefer" when configuring the server.

To display all NTP servers used, run the command:
	**show ntp association**
This displays all NTP servers the device is connected to alongside some meta data. The IP address will appear alongside some symbols:
- '**\***': Non-authoritative time source
- '**+**': candidate for an NTP server
- '**~**': configured NTP server
- '**-**': outlier, won't be used
- '**x**': falseticker, won't be used

The table also contains:
- The reference clock the NTP servers use
- stratum level
- other fields related to NTP
You can issue:
	**show ntp status**
to display general information about the NTP process. If you issue the **show clock detail** command, the timezone used will be UTC. To update the clock time with a specific timezone, first configure the timezone manually (using the command **clock timezone**) and then issue the command:
	**ntp update-calendar**
in global configuration mode to update the internal clock. The internal clock keeps running even after device shutdown and so it is used to initialize the software clock.
### NTP Client & Server mode
When syncing to an NTP server, the device becomes an NTP server itself and can provide time information to other devices. In global configuration mode, you can follow these steps to configure an NTP server:
	**int loopback0**
	**ip address <IP address\> <mask\>**
	**exit**
	**ntp source loopback0**
The creation of a loopback interface is useful as binding it to a specific interface might be problematic if that interface goes down. As such, it is instead bound to a virtual interface on the router itself so that it can be reached from multiple destinations. Issuing the **ntp source** command sends NTP messages from the IP address of the specified interface.
### NTP Server mode
If there is no authoritative NTP server that is reachable by your network, you could still use a local NTP server so that all other devices agree on the same time, even if it is slightly off. To manually configure a Cisco device to operate as an NTP server (without being connected to a lower stratum NTP server) you can issue the command:
	**ntp master \[stratum level 1-15]**
In global configuration mode. If the stratum is not specified, the default stratum of 8 will be set. The command will create an NTP server process bound to a loopback IP address (in 127.0.0.0/8) and running on a virtual interface that is automatically created for the purpose. 

The **ntp master** command accepts the stratum level as an argument that will be used on the device itself, which is one higher than the stratum of the stratum of the NTP process running on the virtual interface. So if the default stratum of 8 is used, the stratum of the time server on the loopback interface will be 7.
### Symmetric Active mode
If the stratum level of the device is equal to the stratum level of another device, they can form peers to get more accurate data, this can be done by issuing the command:
	**ntp peer <ip address of NTP peer\>**
The other NTP server must be of equal stratum level **I assume**. It must probably also needs to be have an NTP association with the device, **I also assume**.
### NTP Authentication
Optional feature of NTP that allows devices to connect to specific intended servers based on an authentication key. To enable this feature, run:
	**ntp authenticate**
	**ntp authentication-key <key-number\> m5 <key\>**
	**ntp trusted-key <key-number\>**
	**ntp server <ip-address\> key <key-number\>**
These commands, respectively:
1. enable NTP authentication
2. Create NTP authentication keys
3. Specify the trusted keys that can be used
4. Specify which key to use for which NTP server (only run on NTP clients, not servers)
