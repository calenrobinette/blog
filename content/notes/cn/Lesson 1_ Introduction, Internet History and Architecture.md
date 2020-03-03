---
title: "Lesson 1: Introduction, Internet History and Architecture"
date: 2020-03-02T20:50:47-05:00
---

# Introduction

## Why Study Computer Networks?

The Internet has grown in leaps and bounds in the past 2 years and it is expected that it will only continue to do so. Continued advancements in network designs will lead to greater societal advancements. Advancing internet research involves facinating multi-disciplinary problems. This, in turn, would lead to high-impact research opportunities.

---

## Internet Architecture

Network Protocols provide structure to the network architecture by organizing the protocols into layers. Each layer offers different services.

Main takeaway. Internet protocols are layered, layers allow each part to be compartmentalized giving the advantages of *scalability, modularity, and flexibility*

Disadvantages are: 
- redundancy
- dependancies on info from other layers violate the purpose of layering
- overhead cost goes up as abstraction increases


The International Organization for Standardization (ISO) proposed the 7 layered Open Systems Interconnection (OSI) model the internet is built on. The Internet architecture combined the application, presentation and session layers to create a model with 5 layers that it uses:

1. Application Layer
  a. Protocols: HTTP (web), SMTP (e-mail), FTP (file transfer), DNS (translate domain name -> IP Address)
  b. Information Packet: Message
  c. Services of this layer dependent on the design of the application
2. Presentation Layer
  a. Formats info Session Layer <-> Application Layer
  b. e.g. video stream from raw data to codec readable, ints from big endian to little endian
3. Session Layer
  a. Manages transport streams of data and syncs them if necessary
  b. e.g. ties together audio and video streams
4. Transport Layer
  a. Protocols: TCP, UDP
  b. Information Packet: Segment
  c. TCP
    1. Connection oriented service to apps on above layers
    2. Guaranteed delivery of messages
    3. Flow control to match speeds of senders/receivers
    4. Congestion-control mechanisms when the network is congested

    d. UDP
      1. Best effort service, not reliable
5. Network layer
  a. Protocols: IP, Routing
  b. Information Packet: Datagram
  c. Delivers Segments from the transport layer on the local host to the transport layer on the destination host
6. Data Link Layer
  a. Protocols: Ethernet, PPP, Wifi
  b. Information Packet: Frame
  c. Responsible to move frames from one node (host or router) to the next node
  d. At each node the Network layer passes the datagram to the data link layer for delivery where the data link layer at that node passes it up to the network layer
7. Physical Layer
  a. Protocols: Many, for ethernet: twisted-pair copper wire, coaxial cable, single-mode fiber optics
  b. Transfers bits within a frame between two nodes that are connected
  
---

## Layers Encapsulation

Message (M) is sent to Transport layer. Transport layer appends transport layer header information (Ht). Both M + Ht is called a *segment.* This continues with the network layer adding it's header information (Hn). Hn + Ht + M is called a *datagram.* This continues to the data link layer adding header info (Hl) to make a frame. At the receiving end the opposite happens where each step involves stripping off the headers at each layer. This is called *de-encapsulation.*

Intermediate devices between hosts may have layer-2 (switches) or layer-3 (routers) devices.

---

## End-to-end principle

"The function in question can completely and correctly be implemented only with the knowledge and help of the application standing at the endpoints of the communications system. Therefore, providing that questioned function as a feature of the communications systems itself is not possible." - *End-to-End Arguments in System Design", Saltzer, Reed, and Clark*

Keep the middle simple so that advancements are not costly. Also, all resources needed for most layer research/advancements are available on hosts and do not require much special access. Main reasoning is that all applications don't need the same features and network functions to support them. Thus the choice to use those functions are kept at the ends of networks design.

**Violations of the e2e principal:** Firewalls and traffic filters. These violate the principle because they can drop end host communication. Network Address Translation (NAT) boxes are required to fix a shortage of IP addresses, but also violate the ideals of e2e. IP addresses are distributed to devices on a subnet, while the NAT box has it's own outward facing IP address accessed by other hosts. It acts as a translation tool to any packets entering or leaving the LAN. It is a violation of e2e because hosts cannot create a direct link to hosts behind the NAT device.

Interesting workarounds:
- STUN: a tool that allows hosts to discover NATs and the public IP address and port number that the NAT has allocated for the application that the host wants to communicate with
- UDP hole punching: establishing a bidirectional UDP connection between hosts behind NATs

---

## The Hourglass Shape of Internet Architecture

Internet protocol stack is an hourglass shape with IP addresses in the middle. Lots of innovation/change/research done to improve the top and bottom layers while the middle hasn't seen many advancements in recent years. How do we seek out improvements to all layers?

---

## Evolutionary Architecture Model (EvoArch)

Researchers have suggested this model to help study layered architectures, and their evolution in a quantitative manner.

Copied from lecture: 

- Layers. A protocol stack is modeled as a directed and acyclic network with L layers.

- Nodes. Each network protocol is represented as a node. The layer of a node u is denoted by l(u).

- Edges. Dependencies between protocols are represented as directed edges.

- Node incoming edges. If a protocol u at layer l uses the service provided by a protocol w at the lower layer l−1, then this is represented by an “upwards” edge from w to u.

- Node substrates. We refer to substrates of a node u, S(u), as the set of nodes that u is using their services. Every node has at least one substrate, except the nodes at the bottom layer.

- Node outgoing edges. The outgoing edges from a node u terminate at the products of u. The products of a node u are represented by P(u).

- Layer generality. Each layer is associated with a probability s(l), which we refer to as layer generality. A node u at layer l+1 selects independently each node of layer l as the substrate with probability s(l). The layer generality decreases as we move to higher layers, and thus protocols at lower layers are more general in terms of their functions or provided services than protocols at higher layers. For example, in the case of the Internet protocol stack, layer 1 is very general and the protocols at this layer offer a very general bit transfer service between two connected points, which most higher layer protocols would use.

- Node evolutionary value. The value of a protocol node, v(u), is computed recursively based on the products of u. By introducing the evolutionary value of each node, the model captures the fact that the value of a protocol u is driven by the values of the protocols that depend on it. For example, let’s consider again the Internet protocol stack. TCP has a high evolutionary value because it is used by many higher layer protocols and some of them being valuable themselves. Let’s assume that we introduce a brand new protocol, at the same layer as TCP, that may have better performance or other great new features. The new protocol’s evolutionary value will be low if it is not used by important or popular higher layer protocols, regardless of the great new features it may have. So the evolutionary value determines if the protocol will survive the competition with other protocols, at the same layer, that offer similar services.    

- Node competitors and competition threshold. We refer to the competitors of a node u, C(u), as the nodes at layer l that share at least a fraction c of node u’s products. We refer to the fraction c, as the competition threshold. So, a node w competes with a node u, if w shares at least a fraction c of u’s products.

- Node death rate. The model has a death and birth process in place, to account for the protocols that cease or get introduced respectively. The competition among nodes becomes more intense, and it is more likely that a protocol u dies if at least one of its competitors has a higher value than itself. When a node u dies, then its products also die, if their only substrate is u.

- Node basic birth process. The model, in its simplest version, has a basic birth process in place, where a new node is assigned randomly to a layer. The number of new nodes at a given time is set to a small fraction (say 1% to 10%) of the total number of nodes in the network at that time. So, the larger a protocol stack is, then the faster it grows.

Summary: EvoArch is a method to emulate the possibilities of new protocols successes/failures. It is done in a step-by-step fashion adding nodes and conntecting them to see if they could be successful. And if a node has been made obsolete by another, better performing node (as in, it services more nodes on other layers.)
