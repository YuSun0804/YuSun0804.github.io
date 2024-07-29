---
title: Linux Network Virtualization
categories:
 - Devops
tags: [Docker Network, Virtualization, Linux]
---

## Introduction
Network virtualization refers to the technology that merges several physical networks into a single virtual network, operated through software, or partitions a single physical network into distinct, autonomous virtual networks. While network virtualization encompasses various aspects, this article specifically delves into its functionality on Linux, examining how individual containers communicate across different hosts.

## Methodologies
### Linux Network Stack
Before delving deeper into Linux Network Virtualization (LNV), let's first examine the network model on Linux. The diagram below illustrates the network stack model in Linux. 

![vm_directory @1x]({{ "/assets/images/post/linux-network-virtualization/network-stack.drawio.svg" | absolute_url }})
#### Socket
The program at the application layer communicates with the network protocol stack in the kernel space via the Socket programming interface. specifically applications interact with sockets through receive and send buffers. Besides, as following the design principle of "everything is a file" in Unix and Linux systems, socket operations are executed as read and write operations to the file system (socketfs) through file descriptors.

#### TCP/UDP
TCP and UDP are the two main transport layer protocols, and from the perspective of how they operate on the network stack, they function similarly. Taking TCP as an example. When new data is copied into the send buffer of the socket, the kernel encapsulates this data as TCP segment packets. These packets, typical of common network protocols, consist of a header and a body (also known as the "payload").

Subsequently, the system kernel extracts the data from the buffer as body segment. In header, it then incorporates necessary control information at the transport layer, such as the source and destination port numbers representing the sending and receiving programs respectively. Additional information like sequence numbers for ensuring reliable communication (used for retransmission and sequencing control) and checksums for verifying data integrity during transmission.

#### IP
The most vital network layer protocol is the Internet Protocol (IP), accompanied by others like the Internet Group Management Protocol (IGMP) and various routing protocols (such as EGP, NHRP, OSPF, IGRP, etc.).

Taking the IP protocol as an example, it accepts packets from the upper layer (for instance, the TCP packet in the preceding example) in packet format. It then appends its own packet header, indicating details like the destination address for routing, packet length, protocol version number, and other pertinent information. This encapsulated data is then formed into IP packets and forwarded to the subsequent layer.

#### Device
The term "Device" in this context refers to a network device, but it's distinct from the conventional notion of physical hardware devices. Instead, it serves as an open interface to the operating system. This interface may represent actual physical hardware or specific program code with defined functions, even in the absence of a physical network card.

The primary role of the Device is to abstract a unified interface, enabling program code to select or influence the packet's sending and receiving path. For instance, it helps determine which network card device should transmit data. Additionally, it facilitates the preparation of data required for network card driver operations, such as the IP packet from the previous layer and the MAC address of the Next Hop (obtained through ARP Requests).

#### Driver
The Network Card Driver (Driver) functions as a hardware-oriented interface within the network access layer. It operates by copying the data packet intended for transmission into the main memory via DMA, placing it in a buffer within the driver (often referred to as a ring buffer). During this process, various information including the IP packet provided by the upper layer, the MAC address of the next hop, and the VLAN Tag are encapsulated into an Ethernet Frame.

### Netfilter
The Netfilter framework, conceptualized and spearheaded by Rusty Russell, the principal maintainer of Linux firewalls and networks, revolves around the network layer, particularly the IP protocol. It incorporates five hooks within this layer. Whenever a packet traverses the network layer and encounters these hooks, the kernel module automatically triggers the registered callback function. This mechanism allows program code to intervene in Linux network communication via the callback function.

![vm_directory @1x]({{ "/assets/images/post/linux-network-virtualization/network-stack.drawio.svg" | absolute_url }})

Here are what these five hooks are: 
* PREROUTING: This hook is triggered as soon as packets from the device enter the stack. Note that if the PREROUTING hook is triggered before entering the IP route, it means that the PRERouting hook will be triggered whenever packets are received, whether they are actually sent to the machine or not. It is generally used for Destination network address translation (DNAT).
* INPUT: After the packet is routed through IP, if it is determined to be sent to the local machine, this hook will be triggered, which is generally used to process packets sent to the local process.
* FORWARD: After the packet is routed through the IP address, if it is determined that the packet is not destined for the local machine, this hook will be triggered. It is generally used to process packets forwarded to other machines.
* OUTPUT: Packets sent from the native program, before the IP route, will trigger this hook, which is generally used to process the output packets of the local process.
* POSTROUTING: Packets that go out of the local network card, whether sent by a local program or forwarded by the local machine to another machine, will trigger this hook, which is generally used for Source NAT (SNAT).

#### iptables
There are many Netfilter-based applications, of which the most widely used are undoubtedly the Xtables family of tools, such as iptables, ebtables, arptables, ip6tables, and so on. If you've ever developed on a Linux system, you probably have at least used the iptables tool, which is often referred to as the "built-in firewall" of Linux.

iptables has five non-extensible rule tables built in (the security table is not commonly used, many data only calculate the first four tables), let's look at:
* raw table: Used to remove Connection Tracking from packets.
* mangle table: It is used to modify the header information Of data packets, such as the Type Of Service (ToS) and Time to Live (TTL), and set the Mark for data packets. The typical application is the Quality Of Service (QoS) management of links.
* nat table: Used to modify information such as the source or destination Address of data packets. The typical application is Network Address Translation.
* filter table: Used to filter packets and control whether packets arriving at a certain chain are allowed, directly discarded or rejected (ACCEPT, DROP, REJECT). The typical application is a firewall.
* security table: Used to apply SELinux to packets, this table is not commonly used.
The five rule tables are prioritized: raw→mangle→nat→filter→security. Note that when you add a rule to iptables, you need to specify which table to store in according to the intent of the rule. If you do not specify, the filter table will be stored by default. In addition, each table can use a different chain, the specific table and chain corresponding relationship is as follows:

![vm_directory @1x]({{ "/assets/images/post/linux-network-virtualization/iptable.png" | absolute_url }})

So, you can actually see from the name, the preset five chains are directly from the Netfilter hook, they and the five rule table corresponding relationship is fixed, the user can not increase the custom table, or modify the relationship between the existing table and the chain, but can increase the custom chain.

The new custom chains have no natural correspondence with Netfilter's hooks, in other words, they are not triggered automatically, and can only be executed by explicitly using the JUMP behavior to jump past the default five chains.

### VNIC
#### tun/tap
tun and tap are two relatively independent virtual network devices. tap simulates an Ethernet device and operates Layer 2 packets (Ethernet frames), while tun simulates a network layer device and operates Layer 3 packets (IP packets). Then, the purpose of using the tun/tap device is to send the packets from the protocol stack to a user process that has opened the /dev/net/tun character device for processing, and then send the packets back to the link. Here you can informally understand that one end of the virtual network card driver is connected to the network protocol stack, the other end is connected to the user state program, and the ordinary network card driver is connected to the network protocol stack at one end, and the other end is connected to the physical network card. In this way, as long as the packet in the protocol stack can be intercepted and processed by the user state program, the programmer has enough stage space to play a variety of tricks, such as data compression, traffic encryption, transparent proxy and other functions, can be implemented on this basis.
#### veth
It is not very accurate to directly compare veth to a virtual network card. If you want to compare it to a physical device, it should be equivalent to a pair of physical network cards connected by a crossover cable. Additional knowledge: Straight line sequence, cross line sequence - The crossover cable refers to the cable with one end of the T568A standard and the other end of the T568B standard. The straight-connected network cable uses the same standard network cable at both ends. - Network card to network card of the same kind of equipment, you need to use a cross-line sequence network cable to connect, network card to the switch, the router uses a direct line sequence network cable, but now most of the network card with the line sequence flip function, direct line can also be connected to the network card to the network card.
veth is not actually a device, but a pair of devices, so it is often referred to as a veth pair. If we want to use veth, it makes sense to do it in two separate network namespaces, because veth pairs are connected to the protocol stack at one end and to each other at the other end. When you enter data on one end of the veth device, the data flows out of the other end of the device unchanged
Although veth solves the communication problem between two containers well by simulating the direct connection of network cards, for communication between multiple containers, if you still only use veth pairs, things will become very troublesome. After all, it is not practical for each container to establish a special veth pair for other containers that communicate with it. It would be expensive to actually do it.

### Linux Bridge
Since there is a virtual network card, it is natural for us to associate the network card with the switch to realize the mutual connection between multiple containers. The Linux Bridge is a virtual Switch under the Linux system, although it is named "Bridge" (Bridge) rather than "switch" (Switch), but in the process of use, you will find that the Linux Bridge looks like a switch, The function is used like a switch and the program is implemented like a switch, so it is actually a virtual switch.

### VLAN
The full name of the VLAN is "Virtual Local Area Network" (Virtual Local Area Network), from the name, it is also one of the early results of network virtualization technology.

Due to the working characteristics of Layer 2 networks, vlans depend on broadcast. Whether it is broadcast frames (such as ARP requests, DHCP, and RIP) or flood routing, the cost of executing a VLAN increases proportionally with the increase in the number of devices connected to Layer 2 networks. It can easily form a Broadcast Radiation storm.

Therefore, the primary responsibility of a VLAN is to divide the broadcast domain and distinguish devices connected to the same physical network.

The VLAN Tag is added to the packet header of an Ethernet frame so that all broadcasts take effect only for devices with the same VLAN Tag. This shrinks the broadcast domain and, as a side effect, improves security and manageability, since the two vlans cannot communicate directly with each other. If there is a need for communication, it must be done through a Layer 3 device, such as a Router on a Stick or a Layer 3 switch.

However, vlans have two obvious drawbacks, the first of which is the design of the VLAN Tag. When the 802.1Q specification for VLAN was first proposed in 1998, network engineers could not possibly have anticipated the popularity of cloud computing in the future, so they only reserved 32 Bits of storage space for VLAN tags. There are also 16 Bits of storage Tag Protocol Identifier (Tag Protocol Identifier), 3 Bits of storage Priority Code Point (Priority Code Point), and 1 Bits of storage standard Format indication (Canonical Format) Indicator), the remaining 12 Bits are used to store the Virtualization Network Identifier (VNI).

In other words, a VLAN ID can only have a maximum of (\mathrm{2}^{12})=4096 values. When the cloud computing data center appears, even without considering the requirements of virtualization, there may be tens of thousands or even hundreds of thousands of physical devices that need to allocate IP, so 4096 vlans are certainly not enough.

Later, IEEE engineers proposed 802.1AQ specification to remedy this defect, the general idea is to attach two VLAN tags to the Ethernet frame in succession, each Tag still only 12 Bits of VLAN ID. However, the two together can store (\mathrm{2}^{24})=16,777,216 different VLAN ids, because the two VLAN tags are placed side by side in the packet header, the 802.1AQ specification also has a nickname for QinQ (802.1Q in 802.1Q).

QinQ was introduced in 2011, but until now it has not been particularly popular, because in addition to requiring device support, it also does not solve the second defect of VLAN: cross-data center transmission.

The VLAN itself is designed for Layer 2 networks, but information can only be transmitted across layers 3 between two independent data centers.

### VxLAN
To solve the above two problems, the IETF defines the VXLAN specification, which is one of the standard technical specifications of Network Virtualization over Layer 3 (NVO3). It is a typical Overlay network.

The VXLAN uses the L2 over L4 (MAC in UDP) packet encapsulation mode to encapsulate Ethernet frames transmitted at Layer 2 into Layer 4 UDP packets, and adds its own VXLAN Header. The VXLAN Header contains 24 Bits of VLAN ID, which can also store 16.77 million different values.

However, VXLAN also introduces additional complexity and performance overhead, as shown in the following two points:
The transmission efficiency decreases. If you carefully count the Bytes of the UDP, IP, and Ethernet headers in the previous VXLAN packets, you will find that the newly added headers of the packets encapsulated by VXLAN account for a total of 50 Bytes (VXLAN headers account for 8 Bytes, and UDP headers account for 8 bytes, respectively). The IP packet header occupies 20 Bytes, and the MAC header of the Ethernet frame occupies 14 Bytes), but originally only 14 Bytes is needed, and now the 14 Bytes are still consumed, but they are blocked in the innermost Ethernet frame. The MTU of Ethernet is 1500 Bytes. If a large amount of data is transmitted, the additional loss of 50 Bytes is not a high cost, but if the data transmitted is only a few Bytes, the cost of transmission is very high.
The transmission performance deteriorates. Each VXLAN packet packet encapsulation and decapsulation is an extra process, especially for VTEP implemented by software. Extra computing resource consumption sometimes becomes a significant performance impact factor.

### MACVLAN
Macvlan allows you to configure multiple virtual network interfaces on one network interface of the host. These network interfaces have their own independent MAC addresses and can also be configured with IP addresses for communication. VMS or container networks in Macvlan share the same broadcast domain with hosts on the same network segment. Macvlan is similar to Bridge, but because it eliminates the need for a Bridge, it is simple to configure and debug, and relatively efficient. In addition, Macvlan itself perfectly supports vlans.

Data transmission between vlans is implemented through Layer 2 access, that is, MAC addresses, and does not require routing. Users of different vlans cannot communicate directly by default, and if they want to communicate, they also need Layer 3 devices to do routing, and the same is true for MACvlans. The virtual network adapter created by Macvlan technology is logically equivalent to the physical network adapter. A physical network adapter is equivalent to a switch that records the corresponding virtual network adapter and MAC address. When a physical network adapter receives a packet, it determines which virtual network adapter the packet belongs to based on the destination MAC address. This means that as long as packets are sent from the Macvlan subinterface (or to the Macvlan subinterface), the physical network card only receives the packets and does not process them, so this leads to a problem: the IP on the local Macvlan card cannot communicate with the IP on the physical network card! The solution to this problem will be discussed in the next section.

### IPVLAN
ipvlan is similar to macvlan with the difference being that the endpoints have the same mac address. ipvlan supports L2 and L3 mode. In ipvlan l2 mode, each endpoint gets the same mac address but different ip address. In ipvlan l3 mode, packets are routed between endpoints, so this gives better scalability.

## Docker Network Drivers