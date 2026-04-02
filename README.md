*This project has been created as part of the 42 curriculum by aluque-v*

## Table of Contents
 
- [Description](#description)
- [Instructions](#instructions)
- [Resources](#resources)
  - [Protocols](#protocols)
  - [IP Addressing](#ip-addressing)
  - [Subnet masks](#subnet-masks)
  - [Subnetting & CIDR](#subnetting-cidr)
  - [Default Gateways](#default-gateways)
  - [Routing tables](#routing-tables)
  - [Routers and switches](#routers-and-switches)
  - [OSI Layers](#osi-layers)
- [Common Pitfalls](#common-pitfalls)
- [Study Materials](#study-materials)

## Description

NetPractice is a project that consists of 10 progressive levels in which you must configure small-scale networks so that all devices can comunicate with each otehr. Every exercise requires understanding of TCP/IP Addressing, subnetting and routing logic.

## Instructions

The execution of the program is very simple.
1. Download the project archive from the project page in the 42 intra
2. Run the ```run.sh``` script.
3. This shell script will launch a web server and open your preferred web browser to the dedicated page.
4. For each level, a non-functioning diagram is displayed. You can click on the **Get my config** to save a .json file with your configuration for the exercise and then **Next Level** to proceed to the next level.
5. Additionally, when trying to succeed in the exercises, a display for logs is shown at the bottom right of your screen.

## Resources

### Protocols

A "Protocol" is a set of rules and guidelines that governs the communication and interaction between different entities in a system. In the context of computer networks, it defines how data is transmitted, recieved, and interpreted between devices.

Protocols can operate at different [layers](#osi-layers) of the network stack. For example, the **Internet Protocol (IP)** operates at the network layer(layer 3) and defines the addressing scheme and routing mechanisms, while the **Transmission Control Protocol (TCP)** operates at the transport layer and provides reliable, connection-oriented data delivery. Another popular protocol is the **User Datagram Protocol (UDP)**. It is a connectionless, lightweight transport layer. UDP is a simple best-effort protocol that offers low overhead and minimal error checking, making it suitable for scenarios where speed and efficiency are prioritized over reliability (online games, for example).

I could talk a lot about these two main protocols, but it goes off the concepts of this project, so we'll leave it for another one.

### IP addressing
Every device on a network is identified by an IPv4 address (we'll leave IPv6 for later)— a 32-bit number written as four octets (e.g. ```192.168.1.1```). Each octet ranges from 0 to 255. IPv4 and IPv6 addresses have two parts: a network portion (identifies the network) and a host portion (identifies the device within that network). The boundary between them is defined by the subnet mask.
  There are also reserved ranges:
```
  - 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 — private addresses (RFC 1918)
  - 127.0.0.0/8 — loopback (localhost)
  - 0.0.0.0 and 255.255.255.255 — special (default route, broadcast)
```
### Subnet masks
A subnet mask is a 32-bit value that tells you which bits of an IP address belong to the network and which to the host. For the sake of simplicity, both IPs and masks are written in a decimal number system so we humans (ew) can easily read them. However, a computer reads this IP: ```192.0.12.68```, like this:
```
1100000.00000000.00001100.01000100
```
Subnet masks, can be written in dotted notation (e.g. 255.255.255.0) or CIDR notation (/24). A bitwise AND between IP and mask gives the network address. The inverse gives the host range. Example:
```
  - IP:         192.168.1.130   =   11000000.10101000.00000001.10000010
  - Mask:       255.255.255.192 =   11111111.11111111.11111111.11000000 (/26)
  - Network:    192.168.1.128   =   11000000.10101000.00000001.10000000
```

The first address in the range is the network address (not assignable), the last is the broadcast address (not assignable). Everything in between are usable host addresses.

### Subnetting & CIDR
#### What exactly are Network Classes?
Network classes refer to an early classification system of networks that divided networks by size into Class A, Class B and Class C. The amount of IP combinations goes from largest to smallest, so Class A has the most combinations. For network classes, IP addresses are divided into a Network ID and a Host ID.

For example, the IP address 192.1.12.70, we can say 192.1.1 is the Network ID, and 70 is the Host ID. The following table shows an IP example for each class:

| **Name**    | **IP Range**                 | **Purpose**                | **Total IP Address Amount** | **Example (Network ID Bolded)** |
|-------------|------------------------------|----------------------------|-----------------------------|---------------------------------|
| **Class A** | 1.0.0.0 to 126.255.255.255   | Large-Sized Organizations  | 2^24 (16.777.216)           | **125.**12.12.12                |
| **Class B** | 128.0.0.0 to 191.255.255.255 | Medium-Sized Organizations | 2^16 (65.536)               | **128.12.**12.12                |
| **Class C** | 192.0.0.0 to 223.255.255.255 | Small-Sized Organizations  | 2^8 (256)                   | **192.18.75.**212               |
| **Class D** | 224.0.0.0 to 239.255.255.255 | Multicast Groups           | **N/A**                     | **N/A**                         |
| **Class E** | 240.0.0.0 to 255.255.255.255 | Experimental               | **N/A**                     | **N/A**                         |


#### CIDR
Subnetting divides a larger network into smaller sub-networks. CIDR (Classless Inter-Domain Routing) replaced the old class-based system (Class A/B/C). The /n notation tells you how many bits are the network prefix.

| CIDR |            Mask | Hosts |
|-----:|----------------:|------:|
|  /24 |   255.255.255.0 |   254 |
|  /25 | 255.255.255.128 |   126 |
|  /26 | 255.255.255.192 |    62 |
|  /27 | 255.255.255.224 |    30 |
|  /28 | 255.255.255.240 |    14 |
|  /29 | 255.255.255.248 |     6 |
|  /30 | 255.255.255.252 |     2 |


  A /30 gives exactly 2 usable hosts — perfect for point-to-point links between routers

#### Subnet calculation walkthrough
  **Problem:** you need a subnet that can hold 20 hosts.

  1. Find the smallest power of 2 that satisfies: 2^n - 2 >= 20 → n = 5 → 2^5 - 2 = 30 usable hosts
  2. Host bits = 5, so network bits = 32 - 5 = 27 → mask is /27 (255.255.255.224)
  3. Pick a base network, e.g. 192.168.1.0/27

  ```
  Network address:    192.168.1.0       (not assignable)
  First usable host:  192.168.1.1
  Last usable host:   192.168.1.30
  Broadcast address:  192.168.1.31      (not assignable)
  Next subnet starts: 192.168.1.32
  ```

  The block size is always 2^(host bits). Here: 2^5 = 32. Subnets in this scheme start at .0, .32, .64, .96, .128, .160, .192, .224.


### Default gateways
  A default gateway is the IP address a device sends packets to when the destination is not on the same subnet. It's typically a router interface. Rules:
  1. The gateway address must be on the same subnet as the device using it
  2. The gateway address must be the IP of an actual router interface
  3. If two devices are on the same subnet, they communicate directly (no gateway needed)
  4. If they're on different subnets, each must have a gateway configured pointing to a router that can reach the other network

### Routing tables
  A routing table is a set of rules that tells a router (or host) where to send packets based on the destination IP. Each entry has two parts:
  - **Destination**: a network address with mask (e.g. `10.0.0.0/8`)
  - **Next hop**: the IP address of the next router to forward the packet to

  Key concepts:
  - **Default route** (`0.0.0.0/0`): matches any destination not covered by a more specific entry. Acts as a catch-all — "if you don't know where to send it, send it here."
  - **Longest prefix match**: when multiple entries match a destination, the router picks the most specific one (highest /n value). For example, a packet to `10.1.1.5` matches both `10.0.0.0/8` and `10.1.1.0/24` — the `/24` wins because it's more specific.

  Example routing table on a router with two interfaces:
  ```
  Destination        Next Hop
  ──────────────────────────────────
  192.168.1.0/24     (directly connected - interface 1)
  10.0.0.0/24        (directly connected - interface 2)
  0.0.0.0/0          192.168.1.1
  ```

### Routers and switches
  - Switch (Layer 2): forwards frames based on MAC addresses within the same network. In NetPractice, switches just connect devices on the same subnet — they don't change
  addressing. No configuration needed.
  - Router (Layer 3): forwards packets between different networks. Each router interface has its own IP address on a different subnet. A router looks at its routing table to
  decide where to send a packet. In NetPractice, you configure routing tables with entries like destination_network/mask -> next_hop_ip.

### OSI layers
The OSI model has 7 layers, but for NetPractice you mainly care about layers 1-3:

| Layer |                             Name |     Unit |                   NetPractice relevance |
|------:|---------------------------------:|---------:|----------------------------------------:|
|     1 |                         Physical |     Bits |             Cables (device connections) |
|     2 |                        Data Link |   Frames |                 Switches, MAC addresses |
|     3 |                          Network |  Packets |    IP Addressing, subnet masks, routing |
|     4 |                        Transport | Segments | TCP/UDP (not configured in the project) |
|   5-7 | Session/Presentation/Application |     Data |                       Not relevant here |

1. Physical Layer: This layer deals with the physical aspects of data transmission, such as electrical and mechanical connections, and the conversion of data into signals for transmission over physical media (e.g., copper cables, fiber optics, wireless signals).
2. Data Link Layer: The data link layer provides reliable point-to-point communication between directly connected network nodes. It handles the framing of data into frames, error detection and correction, flow control, and media access control (MAC) protocols. Ethernet is a well-known data link layer protocol.
3. Network Layer: The network layer is responsible for logical addressing and routing of data packets across multiple networks. It encapsulates data into packets, adds network addresses (e.g., IP addresses), and determines the optimal path for packet delivery. The Internet Protocol (IP) is a fundamental network layer protocol.
4. Transport Layer: The transport layer ensures reliable and transparent data transfer between end systems. It breaks down data from the upper layers into smaller segments, provides error detection and correction, and manages end-to-end communication sessions. The Transmission Control Protocol (TCP) and User Datagram Protocol (UDP) operate at this layer.
5. Session Layer: The session layer establishes, manages, and terminates communication sessions between applications on different network nodes. It facilitates synchronization, checkpointing, and recovery of data exchange.
6. Presentation Layer: The presentation layer is responsible for data representation, encryption, compression, and formatting for the application layer. It ensures that data is presented in a format that is understandable by the receiving application.
7. Application Layer: The application layer is the topmost layer and provides network services directly to the end user or application. It includes protocols like HTTP (Hypertext Transfer Protocol), FTP (File Transfer Protocol), SMTP (Simple Mail Transfer Protocol), and DNS (Domain Name System).

## Common pitfalls
  These are the most frequent mistakes when solving NetPractice levels:

  1. **Assigning the network or broadcast address to a host** — the first IP in a range (network address) and the last (broadcast) are reserved. Only addresses in between are usable.
  2. **Gateway not on the same subnet** — a device's default gateway must be an IP on the same subnet as the device itself. A gateway of 10.0.1.1 is useless to a device on 10.0.0.0/24.
  3. **Missing return route** — packets reach the destination, but there's no route configured for the reply. Both directions must be routable.
  4. **Overlapping subnets on different router interfaces** — each router interface must be on a distinct subnet. Two interfaces on the same subnet creates ambiguity.
  5. **Forgetting the default route** — if a router has no entry matching the destination and no `0.0.0.0/0` fallback, the packet is dropped silently.
  6. **Mask mismatch** — two devices on the same link must use the same subnet mask. If one uses /24 and the other /25, they may not see each other as being on the same network.

## Study materials
  - [Practical Networking — Subnet Masking playlist](https://www.youtube.com/playlist?list=PLIFyRwBY_4bRLmKfP1KnZA6rZbRHtxmXi)
  - [Subnet Calculator — visual tool for calculating ranges](https://www.subnet-calculator.com/)
  - [RFC 1918 — Private Address Space](https://datatracker.ietf.org/doc/html/rfc1918)
  - [Computer Networking Full Course (some parts)— freeCodeCamp (YouTube)](https://www.youtube.com/watch?v=qiQR5rTSshw)
  - [NetPractice walkthrough — 42 community guide (GitHub)](https://github.com/lpaube/NetPractice)
