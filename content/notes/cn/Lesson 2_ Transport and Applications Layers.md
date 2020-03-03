---
title: "Lesson 2: Transport and Applications Layers"
date: 2020-03-02T20:53:47-05:00
---

# Introduction

## Transport layer and TCP (Transmission Control Protocol)

The goal is to learn algorithms that provide reliability, flow control, and congestion control. Also, more recent versions of TCP that are designed for better performance/diverse requirements (Such as predictible latency or sustained throughput).

## Readings

### CUBIC: A New TCP-Friendly High-Speed TCP Variant


BIC-TCP is a method to maximize bandwith usage between two users given up-to-date knowledge of current network traffic/congestion to minimize packet loss. This is done using a binary search algorithm to grow the "window" (number of successive packets sent in bursts) to the midway point between current sending and maximum sending with minimum packet loss. This is repeated on each TCP send to quickly increase data flow and check for changing congestion.

CUBIC is the next form of BIC-TCP

## Multiplexing

TCP translates application requests into network messages by assigning a socket to the requests to be sent from. It also takes incomming network messages and reads which socket they're destined for and delivers them to the appropriate application. This allows for multiple apps to make simultaneous network sends instead of having to wait for other applications to send *and recieve* their messages. Ex: spotify and facebook at the same time, packets must be delivered to the correct server and returned to the correct app.

**Multiplexing:** adding socket information to data chunks (thus becoming segments) and forward to the network layer

**Demultiplexing:** delivering segments to the approprate socket

**Connectionless multiplexing/demultiplexing:** UDP socket identifiers are a destination IP address and a destination port number. Recieving host doesn't care where the mesage comes from as long as the destination port number is one that it has a socket for and can demultiplex to.

**Connection oriented multiplexing/demultiplexing:** TCP sock identifiers are both the source and destination IP address and port. Recieving host has a listening socket that takes in all requests coming from TCP clients. Once it recieves a connection request it *creates a socket* using the sent identifiers and demultiplexes incoming data. This establishes a TCP connection and both hosts can send and recieve data between one another.

Persistent HTTP uses the same connection for multiple messages, non-persistent HTTP creates a new TCP connection for every request and response. This can have negative performance inpacts on 

## Why not use process IDs?

Process IDs are OS specific which would make the protocol OS dependent. Also, a single process can set up multiple channels of communication, so using process IDs would prevent demultiplexing. Using process IDs would prevent the use of a listening port (which increases security?)

## UDP

UDP is unreliable and doesn't require the establishment of a connection befort sending packets. So why use UDP?

It doesn't require extra work from the transport layer (no congestion control, no retransmission if ACK isn't recieved) therefor it *can* be faster. It also doesn't require the maintenence of a connection (eg with buffers). Both of these reasons mean less delays.

Delay sensitive applications that don't care about some potential losses are better suited to using UDP.

| Application | Application-Layer Protocol | Underlying Transport Protocol |
| ----------- | -------------------------- | ----------------------------- |
| Remote file server | NFS | Typically UDP |
| Network management | SNMP | Typically UDP |
| Routing protocol | RIP | Typically UDP |
| Name translation | DNS | Typically UDP |
| Streaming multimedia | proprietary | UDP or TCP |
| Internet telephony | proprietary | UDP or TCP |
| email | SMTP | TCP |
| Remote terminal access | Telnet | TCP |
| Web | HTTP | TCP |
| File tansfer | FTP | TCP |

### UDP packet structure

UDP has a **64 bit header**

1. Source and destination ports
2. Length of the UDP segment (header and data)
3. Checksum (an error checking mechanism). The sender adds the src port, dest port and packet length. If there's an overflow it wraps around. Then performs a 1s compliment (1 -> 0, 0 -> 1). The reciever adds all four 16-bit fields (source port, dest port, packet length, and checksum) and checks if all values are 1. If not there was an error. This will always detect 1-bit errors, but if a 2-bit error is on the "edge" of a number it could go unnoticed.

## TCP Three-Way Handshake

### Connection setup

1. The TCP client sends a segement with no data with SYN bit set to 1. It also generates an initial sequence number (client_isn) and includes it in this initial segment.
2. Server allocates a buffer and other resources for the connection and responds with SYNACK segment also with SYN bit set to 1. Also includes `ack = client_isn+1` and a randomly chosen initial squence number `server_seq`
3. Once the client receives the SYNACK segment it also allocates resources for the connection and sends an acknoledgment with SYN set to 0

### Connection teardown

1. When client wants to end: sends segment with FIN bit set to 1.
2. Server acknowledges and begins closing/shutting down resources.
3. Server sends segment with FIN set to 1, indicating connection is closed.
4. Client sends ack then waits for a time then closes.


---

## Reliable Transmission

TCP devs implimented reliable transmission as a primitive in the transport layer. **TCP guarantees an in-order delivery of the application-layer data without any loss or corruption.**

How is this achieved? One way is for the receiver to send acknowledgements (ACKs) for each packet received. If the sender doesn't receive an ACK for a packet it sent within a certain amount of time (timeout) it considers the packet lost and resends it. This is known as **Automatic Repeat Request or ARQ**

**Stop and Wait ARQ** is when the sender waits for the ACK for each packet sent before sending another packet. This is very bad.

**Go-back-N** is when the receiver drops all packets after a missed packet. When the sender timesout/notices the lack of ACKs it starts sending from the dropped packet onward. This is bad.

**Selective ACK**ing is what TCP uses. When there's a dropped packet, the reciever ACKs all further packets received with the dropped packet credentials, signifying it was dropped. When the sender times out on that packet **or receives three repeated ACKs** it resends the dropped packet. In the meantime the receiver buffers the other packets to be able to reassemble the full message.

## Transmission Control

**Why control the transmission rate?** Many perameters affect actual transmission rates of links vs max transmission rates (no other users, users computer has no way of knowing max rate, receiver is receiving from many other users, etc)

**Where should the transmission control function reside in the network stack?** UDP lets apps decide. However, most apps need access to these controls so it makes sense to put it in the transport layer. It also entails the methods of fairness in using the network. Thus, TCP provides these mechanisms. They are part of ongoing research to increase network efficiency.

## Flow Control

First case is to prevent receiver buffer overflow. If application is slower than rate of packets recieved, packets will be lost due to overflow. Rate control via a variable 'receive window' (receive buffer - TCP data in buffer) helps sender match this rate moment-to-moment. The receiver sends the receive window (`rwnd`) in every ACK it responds with.

`rwnd = RcvBuffer - [LastByteRcvd - LastByteRead]`
`LastByteSent - LastByteAcked <= rwnd`

There can be issues if receiver sends `rwnd = 0` and sender stops sending. If the receiver has nothing more to send, then the sender will never know when the buffer opens back up. TCP resolves this by having the sender send packets with a size of 1 byte when rwnd is 0. That way ACKs eventually come back once the buffer is open.

## Congestion Control

Flow control is also used to reduce congestion in a network. These mechanisms need to be *very* dynamic in order to adapt to users joining/leaving and the creation/destruction of connections.

### Goals of Congestion Control

- Effiency: High total network throughput/utilization
- Fairness: Each user should have fair use of bandwidth
- Low delay: If throughput is maxed, further messages will have delay. Some apps (video conferencing) suffer greatly from delays.
- Fast convergence: See CUBIC paper. Each flow should get to its fair allocation asap.

### E2E vs Network-assisted

For network-assisted congestion control, routers could update users by sending their own packets. If those packets were lost or the network was already very congested, this would prove inefficient.

TCP uses end-to-end congestion control. Hosts infer congestion from network behavior.

Certain routers in modern networks do provide explicit feedback to end-host by using ECN and QCN protocols.

### Signs of congestion

Packet delay due to queues in router buffers. *Increases* in round trip time (inferred via ACKs) can be an indicator. However, packet delay is variable, making this tricky

Packet loss due to routers during high congestion. There are other reasons for packet loss, but those are far more rare. Early TCP used packet loss because it was already being tracked for reliability.
