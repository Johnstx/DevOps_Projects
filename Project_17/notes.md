

### Notes on Technology applied within this projects

## IP Address 

IP stands for internet protocol. An IP address is a unique numerical identifier for any device or network that connects to the internet. It’s a communications tool that is to connect computers/devices across the internet. IP consist of rules and regulations for transmission of packets across a network including routing and addressing. IP ensures that the packets of data that travel across a network arrives at the correct location.
An IP address contains the network’s address, the resource/ host address on the network, and the network mask.

There are two versions of IP addresses commonly used on tte internet, namely IPv4 and IPv6.

**IPv4 Address**
An IPv4 address is written as four sets of decimal numbers separated by periods, e.g., 192.168.0.1. the IPv4 address is a 32-bit, where each set of the four sets of numbers are termed octets, so there are four octets in an IPv4 address. The first three octets represent the network address while the last octet represents the host address, this assignment however depends on the network mask. E.g 

 ![Alt text](<images/IP address.jpg>)

An IP address with /24 means that the network address uses the first three octets (24-bits) of the IP, an IP address with /16 means the network address uses the first two octets (16-bits) of the IP.
This is helpful in calculating how many IP addresses are available for a specific CIDR prefix.
CIDR (Classless Inter-Domain Routing) is a method for representing IP addresses and their associated routing prefix.
192.168.1.0/24 – example of a CIDR and its prefix.


**IPv6 Address**
IPv6 Addresses are a newer versioning, designed to accommodate the growing number of internet-connected devices, uses hexadecimal digits and colons, e.g., 2001:0db8:85a3:0000:0000:8a2e:0370:7334.
Each IP address can send information to other IP addresses through discrete chunks known as packets. Each network packet contain the data being transferred along with a header containing the metadata of the packet.
Common types of IP address includes:
1.	Private IP address
2.	Public IP address
3.	Dynamic IP address
4.	Static IP address
5.	Website IP address





**Subnets**

Subnet or subnetwork is the logical subdivision of an IP network. Subnetting is a network design technique used to divide a single IP network into smaller, more manageable sub-networks, or subnets. It helps in optimizing IP address allocation, improving network efficiency, and enhancing security. Subnetting is commonly used in IPv4 networks.
Key characteristics of subnets include:
1.	**IP Address range**: Each subnet is defined by a specific range of IP addresses. This range includes a network address and a set of host addresses. The network address is the base address of the subnet.
2.	**Subnet Mask**: The subnet mask is a 32-bit number that specifies parts of the IP address used for network and hosts. It consists of consecutive 1s followed by consecutive 0s. The 1s specify the network portion while the 0s specifies the host portion.
3.	**Benefits of Subnetting**:
•	Efficient IP address allocation
•	Reduced broadcast traffic
•	Improved security
4.	**Subnetting example**:
•	An organization with the IP address range of 192.168.1.0 and a subnet mask of 255.255.255.0 (or/24 in CIDR notation) has a subnet with 256 host addresses (from192.168.1.1 to 192.168.1.254).
•	Changing the subnet mask to 255.255.255.128 (or /25 in CIDR notation), the organization can create two subnets, each with 128 host addresses: 192.168.1.0/25 and 19.168.1.128/25.
5.	**Routing**: Subnets are essential for routing within an IP network. Routers use subnet information to determine how to forward data packets to their destination. The subnet mask helps routers make routing decisions based on the destination IP address.
6.	***VLSM (variable Length Subnet Masking):*** VLSM is a subnetting technique that allows for the allocation of subnets with varying sizes within the same IP address range. This is particularly useful for optimizing IP address allocation in complex networks.



**CIDR Notation**

CIDR (Classless Inter-Domain Routing) notation is a way of representing IP address and their associating routing prefix. It is used to allocating and manage IP addresses more efficiently.
In a CIDR notation like 172.168.1.1/24, as said earlier it consist of  
•	IP address - 172.168.1.1 
•	 The routing prefix - /24
The routing prefix is used to indicate how many bits in the network portion of the IP address that is set to 1 in the subnet mask. For a prefix length /24, the subnet mask is 255.255.255.0, /16 is 255.255.0.0, /12 is 255.240.0.0 and /8 is 255.0.0.0
CIDR notation is used for IP address allocation and routing to make network management more flexible and efficient. It allows creation of subnets of varying sizes, which can help in optimizing address space and routing in large networks.


**IP Routing**

**IP routing** is an important process in computer networking that involves creating of paths for data packets to travel from a source device to a destination device across an interconnected network. Within a network, routing ensures that there is data delivery, connecting networks and also manages the flow of information within and between networks.

**Internet Gateways**

An **internet gateway** is a network node that connects two different networks that use different rules for communicating. Its serves as a point of connection and control, enabling devices within a private netwoek to access the internet and facilitating the bidirectional flow of data between the LAN and the external internet.
Gateways can perform a variety of tasks like 
Connection to the internet, IP address translation, Security and firewall, Packet forwarding, DHCP and DNS services etc.

 
**NAT**

**Network address translation** is a technique applied in networking that allow multiple devices/servers within a local network to share a single public IP address which is used to access the internet.. This practice helps to conserve the limited pool of available IPv4 address while enabling private networks to communicate with the global internet. 
Organizations use NAT technique to map multiple private addresses inside a local network to a public IP address before transferring the information onto the internet.
TYPES OF NAT includes
Static NAT: Here the NAT uses the same public IP each the private IP/s are translated. This means there will be a consistent public IP associated with the NAT device
Dynamic NAT: Here, NAT chooses random public IP from  a pool each there is private IP translation, this will result to a different public IP each time the NAT device is called up. 
PAT: Port Address Translation is a type of dynamic NAT that bands several local IP addresses to a singular public IP. This is mainly employed in organizations that want all their employees to work under a singular IP address. 


**OSI model**

Open system interconnection (OSI) model is a series of layers which computer systems use to communicate. The OSI Model can be seen as a universal language for computer networking. It is based on the concept of splitting up a communication system into seven abstract layers, each one stacked upon the last.
The layers in the OSI model are
1.	Application layer
2.	Presentation layer
3.	Session layer
4.	Transport layer
5.	Network layer
6.	Datalink layer
7.	Physical layer
Each layer of the OSI Model handles a specific job and communicates with the layers above and below itself.
In order for human-readable information to be transferred over a network from one device to another, the data must travel down the seven layers of the OSI Model on the sending device and then travel up the seven layers on the receiving end.


**TCP/IP – Transmission Control Protocol/ Internet Protocol**

This is a suite of networking protocols that forms the backbone of the internet and most local area networks. It provides a standardized set of rules and conventions for devices to communicate and share data over network. The TCP/IP model is an alternative model of how the internet works. It divides the layers into 4 instead of 7 like the OSI model.
**Components of TCP/IP and their protocols are –** 
1.	Application layer (Layer 7) – Provides network services directly to applications. 
Protocols are HTTP, FTP, SMTP and DNS
Use – Allows software applications on different devices to communicate with each other over a network. 

2.	Transport Layer (Layer 4) – Responsible for end-to-end communication, error detection, and flow control. 
Protocols are 
•	TCP (Transmission Control Protocol) – Ensures reliable and ordered data deliver. Establishes and manages connections and handles flow control.
•	UDP (User Datagram Protocol) – Provides a connectionless and lightweight communication method. 
The transport layer on the TCP/IP model corresponds to the transport layer on the OSI model.
3.	Internet Layer (Layer 3) – The internet layer deals with the logical addressing (IP addressing) and routing. It enables data packets to travel through different networks and router.
Protocol - Internet Protocol (IP) - Provides and end-to-end logical addressing.

4.	Network Access Layer (Layers 1 & 2) – Combines processes of layers 1 and 2 in the OSI model.



**Assume role Policy and Role Policy**

In AWS,`` Roles`` are **identities** that have specific permissions. Roles are assumed by users, applications and services, this is done by running the API action ``sts:AssumeRole``. Once assumed, the identity of the users, applications or services becomes that role and gain the permissions set in the role and may go forward to access an AWS resource (e.g., an S3 bucket). 
``Assume role policy`` are **rules/conditions** that determines the users, applications or services that can **assume** an ``IAM role``. Assume role policy is associated with the IAM role itself and defines who can assume the role based on conditions and permissions
While the ``Role policy`` defines is the **permission and access rights** associated with an IAM role and species the actions the role is allowed to perform on which AWS resources.




