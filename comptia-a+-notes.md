# CompTIA A+ Notes
## Mobile devices
### Mobile Device Networking
#### Mobile E-mail
| Protocol | Original Port | Secure Port |
| --- | --- | --- |
| SMTP | 25 | 465 or 587 |
| POP3 | 110 | 995 |
| IMAP | 143 | 993 |

#### Radio and ID
- History
  - Cell phones originally used GSM (Global System for Mobile communciations) for voice calls, and GPRS (General Packet Radio Service) for transmitting data (2G speeds).
  - Then, UMTC (Universal Mobile Telecommunications System) and EDGE (Enhanced Data-rates for GSM Evolution) was used to attain 3G speeds.
  - 4G and LTE speeds are possible with IMT-Advanced (International Mobile Telecommunications - Advanced) requirements.
  - 5G is currently being deployed
	- ITU IMT-2020 Standard
	- 20 Gbps Maximum
- PRL (Preferred Roaming List) updates:
  - A DB of service provider radio information.
  - Used with CDMA (Code Division Multiple Access) networks.
- Baseband updates:
  - Used with GSM.
  - Also called radio firmware.
- Acronyms
  - IMEI (International Mobile Station Equipment Identity): used to identify the device.
  - IMSI (International Mobile Subscriber Identity): used to identify the user.

## Networking
### Ports & Protocols
#### Introduction to Networking
- TCP/IP: Transmission Control Protocol / Internet Protocol
- Retrieve network info
  - Windows: ipconfig /all
  - Mac OSX: ifconfig
  - Linux: ip a
- Linux commands
  - speedtest cli 
  - nload

#### TCP vs. UDP
- TCP (Transmission Control Protocol)
  - Examples: HTTP, FTP, POP3
  - Show active connections: netstat -n
  - To query DNS: nslookup
    - Domain name lookup: nslookup example.com
    - Reverse DNS lookup: nslookup 192.0.2.1
    - Query a specific type of DNS record: nslookup -type=MX example.com
    - Domain name lookup against a specific DNS server: nslookup example.com 8.8.8.8
- UDP (User Datagram Protocol)
  - Connectionless
  - Non-guaranteed
  - Used by BitTorrents, tunneling protocols, and some streaming media
  - Show active connections: netstat -an
- Should reference IANA port number registry

#### HTTP & HTTPS
| Protocol | Full Name | Port |
| --- | --- | --- |
| HTTP | Hypertext Transfer Protocol | 80 |
| HTTPS | Hypertext Transfer Protocol Secure | 443 |

#### Email Protocols
| Protocol | Full Name | Port | Secure Port | Used For |
| --- | --- | --- | --- | --- |
| SMTP | Simple Mail Transfer Protocol | 25 | 587 or 465 | Sending email |
| POP3 | Post Office Protocol Version 3 | 110 | 995 | Receiving email; downloads emails to a single device, removing them from the server |
| IMAP | Internet Message Access Protocol | 143 | 993 | Receiving email; synchronizes emails across multiple devices by keeping them on the server

#### FTP, SSH, and Telnet
| Protocol | Full Name | Port |
| --- | --- | --- |
| FTP | File Transfer Protocol | 21 |
| FTPS | File Transfer Protocol Secure | 989/990 |
| SSH | Secure Shell | 22 |
| Telnet | Telecommunications Network | 23 |

#### DHCP (Dynamic Host Configuration Protocol)
- Gives out IP addresses to computers on the network
- Ports and flow:
![DHCP sequence diagram](./dhcp-sequence.excalidraw.png)

#### DNS (Domain Name System)
- Accepts requests on port 53

#### LDAP & RDP
| Protocol | Full Name | Port | Secure Port | Notes |
| --- | --- | --- | --- | --- |
| LDAP | Lightweight Directory Access Protocol | 389 | 636 | Used to access and distribute directories of information |
| RDP | Remote Desktop Protocol | 3389 | 443 and others | Used for full or limited control of a remote system |

SMB, CIFS, AFP, and SNMP
- Server Message Block (SMB) provides access to shared resources on Windows networks.
	- SMBs are actual packets that authenticate remote computers
	- SMB can communicate via TCP over port 445 or NetBIOS over port 137-139
	- SMB was also referred to as the Common Internet File System (CIFS) because SMB (SAMBA) would run over CIFS port 3020
- Apple Filing Protocol (AFP) offers files services over the network for Mac computers running OS X
	- Uses port 548 (and sometimes 427)
		- 427 is also used by the Service Location Protocol (SLP)
- Simple Network Management Protocol (SNMP) is used to manage and monitor network devices
	- Uses ports 161 and 162

### Network Devices
#### Switches
- Connects computers on a LAN together via ethernet cables
- Known as a star topology
- Delivers frames of data to the correct port by the MAC (Media Access Control) address of each computer
- Passes information to the correct port based on a switching matrix
- Uses the 802.3 ethernet standards
	- 802.3ab is 1000 Mbps
	- 802.3an is 10 Gbps

#### Routers
- Used to connect computers to other networks
- Can be used as a connection to the internet, or on LANs and WANs
- Routes or sends packets from one location to another
- Does this based on the IP addresses of other routers
- Ethernet address to serial address translation is known as NAT (Network Address Translation)

#### Access Points
- WAP (Wireless Access Point) is a device that allows network connectivity using Wi-Fi
- Commonly used standards include
	- 802.11g = 54/108 Mbps
	- 802.11n = 300/600 Mbps
	- 802.11ac = 1 Gbps
- Transmit data via radio waves on either 2.4 GHz or 5 GHz ranges
	- Channel range 1 - 11 corresponds to 2.4 GHz
	- Channel range 36 - 165 corresponds to 5 GHz
- Security
	- Version
		- WPA2 (Wi-Fi Protected Access version 2)
	- Encryption
		- AES (Advanced Encryption Standard)
	- PSK (Pre-Shared Key) is basically the Wi-Fi password

#### Firewalls
- A device or application designed to protect a network or individual computer from intrusion
- Two types:
	- Personal software firewall
	- Network-based firewall
- Includes
	- NAT filtering ports by matching incoming and outgoing traffic
	- Packet filtering by accepting or rejecting packets by configured rules
		- Most common is SPI (Stateful Packet Inspection)

#### Ethernet-based Devices
- Patch panel
	- Physical device that you connect the cables to for the computers for longer runs of cables
	- Has ethernet ports in the front that connect to individual cables in the back
	- Used for cable management
- Repeater
	- Used to repeat signals over long runs of ethernet cable
- PoE (Power over Ethernet)
	- IEEE 802.3af standard
	- Allows devices to receive both power and data over the same ethernet cable

#### SDN
- SDN (Software Defined Networking) is a type of centralized network management that is dynamic and programmable
- Separates network devices' transmitted data and administration
- Uses a separate system known as a controller
- Broken down into two sections
	- Data plane
		- Where the packets flow
		- Packets can be manipulated by the control plane
	- Control plane
- A common SDN protocol is OpenFlow
