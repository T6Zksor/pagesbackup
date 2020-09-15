# Computer Networking
2020-09-15

Computer Networking - A Top Down Approach, 7th
====================

Chapter 1. Computer networks and the Internet
====================

1.2 The Network Edge
--------------------

+ end systems
    - sit at the edge of the Internet
    - also referred to as hosts, because they host applications

1.2.1 Access Networks
--------------------
+ access networks
    - the network physically connects an end system to the first router(also known as edge router)

Chapter 2. Application Layer
====================

2.1 Principles of Network Applications
--------------------

+ network application architectures
    - Client/Server
    - Peer to Peer

+ client and server processes
    - client, the process initiates the communication
    - server, the process waits to be contacted to begin the session

+ socket
    - software interface
    - API between the application and the network
    - developer has little control of the transport-layer side of the socket

+ addressing processes
    - IP for hosts
    - port number for processes

+ transport services available to applications
    - reliable data transfer, also there are unreliable protocols
    - throughput
    - timing
    - security

+ TCP
    - connection-oriented service
    - reliable data transfer service

+ UDP
    - connectionless
    - unreliable

2.2 The Web and HTTP
--------------------

+ HTTP
    - use TCP as underlying transport protocol
    - stateless protocol
    - HTTP with non-persistent and persistent connections

+ HTTP message format
    - 2 types, request and response

+ HTTP request message format
    - \r\n as separator
    - first line is request line
    - then header lines
    - then blank line
    - then entity body

+ HTTP response message format
    - status line
    - header lines
    - entity body

+ cookie
    - has 4 components
        - cookie header line in HTTP response
        - cookie header line in HTTP request
        - cookie file in end system managed by browser
        - back-end database at the website

+ web caching
    - also called a proxy server
    - CDN, content distribution network
    - conditional GET
        - header If-Modified-Since
        - header Last-Modified
        - HTTP/1.1 304 Not Modified

2.3 Electronic Mail in the Internet
--------------------

+ protocols used between mail servers
    - SMTP

+ mail access protocols
    - POP3, extremely simple
    - IMAP
    - HTTP, web-based e-mail

2.4 DNS
--------------------

+ DNS protocol
    - distributed databases
    - application layer protocol
    - runs over UDP, port 53
    - additional functionality
        - host aliasing, canonical hostname
        - mail server aliasing
        - load distribution

+ DNS servers
    - distributed, hierarchical servers
    - 3 classes
        - root DNS servers
        - top-level domain(TLD) servers
        - authoritative DNS servers
    - local DNS server, forward query into the DNS hierarchy
    - query
        - recursive queries
        - iterative queries
    - DNS caching

+ DNS records
    - resource records
    - type
        - A, standard hostname, value is IP
        - NS, domain, value is hostname of authoritative DNS server
        - CNAME, canonical name
        - MX, canonical name of mail server

+ DNS vulnerabilities
    - DDoS, a deluge of DNS query
    - DNS poisoning

2.5 Peer-to-Peer file distribution
--------------------
(not too much to say)

2.6 Video Streaming and CDN
--------------------

+ DASH

+ CDN
    - why CDN
        - clients too far from data center
        - same bytes sent again and again to same network
        - single point of failure

(skip)

2.7 Socket programming
--------------------

+ UDP
+ TCP

Chapter 3. Transport Layer
====================

3.1 Introduction and Transport-Layer Services
--------------------

+ transport layer
    - provide logical communication
    - implemented in end systems, not network routers
    - extends network layer's delivery services from host-to-host to process-to-process

+ UDP
    - delivery segments
    - simple error checking

+ TCP
    - reliable data transfer
    - congestion control

3.2 Multiplexing and Demultiplexing
--------------------

+ demultiplexing
    - transport-layer segments with some fields to identify the receiving socket

+ multiplexing
    - gathering data chunks from different sockets then pass the segments to the network layer

+ connectionless multiplexing and demultiplexing
    - UDP
    - process identified by a two-tuple (dest IP, dest port)

+ connection-oriented multiplexing and demultiplexing
    - TCP
    - process identified by a four-tuple (source IP, source port, dest IP, dest port), for simultaneous connections

3.3 Connectionless Transport: UDP
--------------------

+ why UDP
    - finer application level control over how data sent
        - TCP has congestion control
        - real-time applications
    - no connection establishment, no delay
    - no connection state, less system resources usage
    - small packet header overhead
        - TCP 20 bytes, UDP 8 bytes

+ running multimedia applications over UDP is controversial
    - no congestion control, network would be high stressed, crowding out normal TCP traffics

+ reliable data transfer over UDP
    - reliability done by application itself, google's QUIC

3.4 Principles of Reliable Data Transfer
--------------------

+ the layer below reliable data transfer protocol may be unreliable

+ rdt 1.0, rdt means reliable data transfer, is a toy protocol
    - use finite-state machine(FSM) definitions
    - data transfer over a perfectly reliable channel

+ rdt 2.0
    - with bit errors
    - ARQ(automatic repeat request) protocols
        - error detection
        - receiver feedback, with positive(ACK)/negative(NAK) packets
        - retransmission
    - stop-and-wait protocols
    - fatal, what if ACK/NAK is corrupted

+ rdt 2.1, handle ACK/NAK corrupted
    - resent packet if checksum of ACK/NAK isn't match
    - add sequence number to packet header to detect retransmission

+ rdt 2.2, send ACK/NAK with sequence number

+ rdt 3.0, over a lossy channel with bit errors
    - how to detect packet loss, and what to do
        - detect on sender side
        - if no reply coming from receiver, after wait long enough, retransmit the packet
        - how long should sender wait? at least round-trip delay
        - may have duplicate data packets, handled by packet sequence number in rdt 2.2
        - countdown timer

+ pipelined transfer protocol
    - the range of sequence number must increase
    - sender and receiver may have more buffer
    - error recovery
        - Go-Back-N
        - selective repeat

+ Go-Back-N protocol
    - window size, N
    - sliding-window protocol
    - sequence number category
        - already ACKed
        - sent, not ACKed
        - usable, not sent yet
        - not usable
    - three types of events
        - invocation from above, when window is full, buffered requests or have a synchronization mechanism
        - receipt of ACK, cumulative acknowledgement
        - a timeout event, resend all packets
    - receiver
        - ACK n-th packet when (n-1)th packet is received
        - discard out of order packets, for simplicity, so receiver don't need buffer out of order packets

+ selective repeat(SR)
    - a single packet error may cause GBN retransmit large number of packets, especially window size is large
    - receiver buffer out of order packets
    - receiver ACK every packet even if it's not in order
    - sender
        - each packet has its timer, resend packet if it is timeout
    - receiver
        - have a receiver window size
        - receive a packet isn't window's lower bound, buffer it, otherwise move forward the window
        - re ACK packets even its ACKed, with certain number below the window's lower bound
    - window size must be less than or equal to half of the maximum sequence number
        - otherwise receiver can't know a packet is a new packet or a retransmission

+ packets arrive out of order
    - sender assume packet can't live longer than a maximum lifetime

3.5 Connection-Oriented Transport: TCP
--------------------

+ TCP connection
    - a logical connection
    - full duplex
    - three-way handshake

+ TCP send data
    - send buffer
    - MSS, maximum segment size, maximum payload size a TCP segment can hold
    - MTU, maximum transmission unit, largest link-layer frame size

+ TCP receive data
    - receive buffer

+ TCP segment structure
    - (skip)

+ sequence numbers and acknowledgement numbers
    - TCP views data as ordered stream of bytes
    - sequence number is byte number of all bytes to send
    - ACK number is next byte number except to receive

+ estimate RTT (round-trip time)
    - SampleRTT for latest mesurement
    - EstimatedRTT for weighted average RTT
    - DevRTT for difference between SampleRTT and EstimatedRTT
    - SampleRTT fluctuates, EstimatedRTT smooth it
    - DevRTT as margin for fluctuations
    - TCP's timeout is EstimatedRTT + 4 * DevRTT

+ doubling the timeout interval
    - if timeout, set next timeout as twice

+ fast retransmit
    - receiver detects missing segments, gap in data stream, then it ACK for missing segments immediately
    - sender receive same ACKs repeatedly, knowing that segment lost, retransmit it, even before timeout

+ flow control
    - a speed matching service
    - sender maintain a receive window, rwnd, means how much the spare buffer size is on the receiver
    - receiver tell rwnd every time it send segments back to sender
    - sender calculate through ACKed sequence number and sent sequence number
    - if rwnd = 0, sender still keep sending small(1 byte) segments, so receiver can ACK, in which contain new rwnd
    - UDP doesn't have this, if buffer overflow, following segments would be dropped

+ connection management
    - three way handshake
    - TCP state transition diagram
    - SYN cookie, prevent SYN flood attack

3.6 Principles of Congestion Control
--------------------

+ cause and cost
    - large queuing delays as packet arrival rate near the link capacity
    - large delays cause sender to retransmit, which is unneeded, and make link more congested
    - packet dropping during congestion, which wasted the forward work of upstream routers

+ approaches
    - end-to-end congestion control, without network layer's assistance
    - network-assisted congestion control

3.7 TCP Congestion Control
--------------------

+ limit sender send rate
    - if little congestion, increase send rate
    - if congestion, decrease send rate

+ cwnd
    - cwnd size is how much data sender send in every RTT

+ congestion control algorithm(TCP reno)
    - slow start
        - add 1 MSS to cwnd after receive per ACK, which means doubling cwnd every RTT, exponential growth
        - if loss event, set ssthresh to cwnd/2, then set cwnd to init value (1 MSS)
        - if cwnd meet ssthresh, enter congestion avoidance mode
    - congestion avoidance
        - cwnd increase 1 MSS per RTT, linear growth
    - fast recovery
        - retransmit when receive 3 duplicated ACK
        - TCP reno use fast recovery

+ retrospective
    - an additive-increase, multiplicative-decrease(AIMD) algorithm
    - alternative congestion control algorithms

+ average throughput
    - is relate to window size and RTT

+ high bandwidth links
    - maximum transfer speed is limited because of congestion control

+ fairness
    - two links with same window size and RTT, will finally come to same transfer rate
    - links with different RTT, smaller RTT will get higher throughput
    - UDP doesn't have congestion control, so UDP traffic will crowd out TCP traffic, unfair
    - parallel TCP connections is unfair, too

+ Explicit Congestion Control(ECN)
    - network layer assisted

Chapter 4. The Network Layer: Data Plane
====================

4.1 Overview of Network Layer
--------------------

+ network layer functions
    - forwarding, in data plane
        - fast, typically hardware implemented
        - forwarding table
    - routing, in control plane
        - slow, often software implemented

+ SDN

4.2 What's Inside a Router
--------------------

+ four components
    - input ports
    - switching fabric
    - output ports
    - routing processor

+ forwarding
    - destination-based forwarding
    - generalized forwarding

+ input port processing
    - forwarding by prefix of destination address
    - longest prefix matching rule
    - lookup is hardware implemented, which is super fast

+ switching
    - via memory
    - via a bus
    - via an interconnection network

+ output port processing

(skip)

4.3 IPv4, Addressing, IPv6, and More
--------------------

+ IPv4 datagram format
    - data length, rarely larger than 1500 bytes because of Ethernet frame size
    - protocol, indicate which protocol used in payload
    - source and destination IP addresses

+ IPv4 datagram fragmentation
    - MTU, maximum transmission transmission, largest link-layer frame size
    - IP datagram may transfer to different link-layer protocol with smaller MTU, where fragmentation happend
    - each IP datagram identified by source address, dest address, identifier in header, offset in header

+ IPv4 addressing
    - IP address is associated with an interface, not a host, a host can have multiple IP address
    - address format, 32 bits long
    - subnet, subnet mask
    - CIDR, classless interdomain routing, identify subnet by prefix
    - ICANN, organization who allocate IP addresses

+ DHCP, dynamic host configuration protocol
    - obtain IP address automatically, also additional infos like, default gateway(first hop router), local DNS, subnet mask
    - four steps
        - DHCP server discovery
        - DHCP server offer
        - DHCP request
        - DHCP ACK

+ NAT

+ IPv6
    - important changes
        - expanded address capabilities
            - 128 bits
            - anycast
        - compact header
        - flow
    - difference with IPv4
        - fragmentation, IPv6 not allow intermediate routers do fragmentation
            - if MTU changed, router drop that datagram and send back "packet too big" ICMP message
            - speed up forwarding
        - header checksum, IPv4 without it
            - don't need recalc after TTL changed, speed up
        - options, so header size is fixed

+ transition from IPv4 to IPv6
    - tunneling, whole IPv6 datagram as payload of IPv4

4.4 Generalized Forwarding and SDN
--------------------

+ OpenFlow

(skip)

Chapter 5. The Network Layer: Control Plane
====================

5.1 Introduction
--------------------

+ per-router control
+ logically centrialized control

5.2 Routing Algorithms
--------------------

+ classify
    - centrialized or decentrialized
    - static or dynamic
    - load-sensitive or load-insensitive

+ link-state algorithm
    - centrialized routing algorithm
    - Dijkstra

+ distance-vector algorithm
    - decentrialized routing algorithm

(skip)

5.3 OSPF
--------------------

+ AS, autonomouse system
    - a group of routers under same administrative control

+ OSPF, Open Shortest Path First
    - intra-AS routing protocol

5.4 BGP
--------------------

+ BGP
    - routing among ISPs

+ IP anycast

(skip)

5.6 ICMP, Internet Control Message Protocol
--------------------

+ function
    - error report
    - ping

+ traceroute
    - sending ICMP messages with TTL 1,2,3,...,etc, to a unreachable port number
    - routers in path send back ICMP warning, destination send back ICMP unreachable

(skip)

(Chapter 6 & 7 read Chinese edition)

Chapter 8. Security in Computer Networks
====================

8.1 What is Network Security
--------------------

+ secure communication
    - confidentiality
    - message integrity
    - end-point authentication
    - operational security

8.2 Principles of Cryptography
--------------------

+ symmetric key cryptography
    - block ciphers
        - DES
        - 3DES
        - AES
    - cipher block chaining

+ public key encryption
    - RSA

8.3 Message Integrity and Digital Signatures
--------------------

+ cryptographic hash function
    - MD5
    - SHA-1

(skip)
(done)
