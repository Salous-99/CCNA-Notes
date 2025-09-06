Port security is a feature on Cisco switches that allows the admin to control what MAC addresses can communicate over the network, also what port (on the switch) is tied to which MAC address, if an unauthorized MAC address connects to a switchport, an action will be taken; by default, the switch will put that port in the *err-disable* state.

 By default, if port security is enabled, a maximum of one MAC address is allowed on the interface, but that number can be increased, a sensible scenario in which you may want to increase the number of MAC addresses allowed on a port is when you have an IP phone connected in this configuration:![[Pasted image 20250905152553.png]]
Since PC traffic is also routed through the same switchport, two addresses must be allowed.

The allowed addresses can be manually configured on the interface, dynamically learned (first MAC address to communicate on the port will be added to the list) or a mix of both.

This is not a perfect solution, security-wise, as an attacker can still spoof their MAC address to communicate. Port security however does afford the admin the ability to limit the total number of MAC addresses that are communicating over the network. Enabling port security would make the DHCP Exhaustion Attack (see [[Security Fundamentals]]) impossible as the attacker PC will be connected to one switchport which will not accept the DHCP Discover frames from thousands of spoofed MAC addresses. However, the MAC address table of the switch can still be filled this way (sending thousands of messages from different spoofed MAC addresses) so again: not perfect.

The default violation mode is **shutdown** but there are other modes, here are all of them:
- Shutdown:
	- places port in err-disabled state
	- generates Syslog/SNMP message when interface is disabled
	- Violation counter is set to 1 when port is disabled
- Restrict:
	- Switch discards traffic from unauthorized MAC addresses
	- Interface is not disabled
	- Each time an unauthorized MAC address is detected, a Syslog/SNMP message is sent
	- The violation counter is incremented by 1 each violation
- Protect:
	- Switch discards frames from unauthorized MAC addresses
	- Interface is not disabled
	- Syslog/SNMP messages not sent for unauthorized traffic
	- Violation counter not incremented
# Secure MAC address Aging
By default, all dynamically learned MAC addresses eventually age out and expire, when running the command **show port-security interface <interface\>**, three fields related to aging will be displayed, these are:
- Aging time: time in minutes before a MAC address ages out
- Aging type: can be either:
	- Absolute: timer starts from the moment the MAC address is learned and expires after the timer finishes. If host is still connected, MAC address is immediately relearned and timer starts again
	- Inactivity: timer starts from the moment the MAC address is learned and is reset every time the port receives a frame with that source MAC address.
- Secure Static Address Aging: Disabled by default, means that statically configured MAC addresses do not age.
# Cisco IOS Configuration
 Port security can only be enabled on switchports in trunk or access mode; desirable and auto are not accepted. After setting the port to either trunk or access, run this command from interface configuration mode:
	 **switchport port-security**
This will enable port-security with the default settings which can be viewed by:
	**show port-security interface <interface\>**

**Note:** running **show port-security** without specifying an interface will display all ports that have port-security enabled and some related information about port-security.

The default violation mode is shutdown which disables the interface. To re-enable it, you can either do it manually:
	**shut**
	**no shut**
from interface configuration mode, **OR** you can enable dynamic error recovery. By default, every 5 minutes (300 seconds) all interfaces that are in the err-disabled state (many reasons this can happen) will become re-enabled if error recovery is enabled for that particular cause. For port security, you can run the command:
	**errdisable recovery cause psecure-violation**
From global configuration mode. The **psecure-violation** refers specifically to the situation in which a port becomes disabled due to port security violation, but this can be any other cause as well. To view all possible causes, run the command:
	**show errdisable recovery**
This will show all possible reasons as to why a port might transition to the err-disable state. You can also change the interval length for re-enabling error disabled ports via the command:
	**errdisable recovery interval <time in seconds\>**
from global configuration mode.

**Note:** Make sure to disconnect the unauthorized device before the port is re-enabled, otherwise, the same thing will happen again.

To pick different violation mode, first enable port security normally, then enter the command:
	**switchport port-security violation {restrict | protect}**

You can statically/manually configure a MAC address using:
	**switchport port-security mac-address <MAC address - dot separated\>**

To configure the aging type, run:
	**switchport port-security aging type {absolute | inactivity}**
To configure aging static addresses, run:
	**switchport port-security aging static**
from interface configuration mode.
## Sticky Secure MAC Addresses
If you issue the command:
	**switchport port-security mac-address sticky**
all dynamically learned MAC addresses will be added to the running-config and will **not age out** even if static aging is enabled. However, these MAC addresses will be saved to the running config, not the startup config so to make them truly permanent you need to write them to memory. If you issue the:
	**no switchport port-security mac-address sticky**
command, all current sticky addresses will revert back to dynamically learned addresses.

running:
	**show mac address-table secure**
will display all secure MAC addresses; stick and static address will have a type of **STATIC**, dynamically learned MAC addresses will have a type of **DYNAMIC**