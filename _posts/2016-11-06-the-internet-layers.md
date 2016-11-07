---
layout: post
title:  "The Internet Layers"
date:   2016-11-06 09:55:32 -0500
category: apprenticeship
tags: [scala]
---

If you ever tried to understand how your computer connects to *the internet*, you probably learned that a **client** &mdash; usually, but not always, a browser &mdash; sends a request to a **server**,  located at the address you typed in in the browser's address bar; then the server sends back a response to the client. That's is pretty accurate. But there is much more to that.<!--more-->

![Client Server Model](/images/client-server.png)

It is possible to describe how that connection happens &mdash; what goes behind the arrows showing the **request** and **response** process &mdash; in many ways. This post will focus on the **layers** traversed by requests and responses in the [Internet Protocol Suite](https://en.wikipedia.org/wiki/Internet_protocol_suite) (**TCP/IP** - Transmission Control Protocol/Internet Protocol).

## Layer by layer

The layers are are organized in such a way that one only communicates to the other immediately before or after. So, if we had 3 layers, this would be what would happen:

```
|---------| -----> |---------| -----> |---------|
| layer 1 | <----- | layer 2 | <----- | layer 3 |
|---------|        |---------|        |---------|
```

Layer 1 talks to Layer 2; Layer 2 talks to 1 and 3, and Layer 3 talks to 2.

Each layer follows certain **protocols** that allow them the communicate and exchange the data between each other.

The TCP/IP has 4 layers and this is how it goes:

1. **The application layer**: The application client

    The **application** layer on both sides &mdash; client and server &mdash; is responsible to follow the [HTTP protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), that establishes a set of rules that client and server use in order to be able to communicate with each other. The HTTP protocol is not the only one that must be handled at the application layer. There are [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol), [FTP](https://en.wikipedia.org/wiki/File_Transfer_Protocol) and others.

    Assuming that the client makes a [GET request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.3) following the HTTP Protocol, the data that is going to be passed to the next layer will hold all the information necessary to establish the communication with the server. It will have a header containing the method (GET), the request [URI](https://tools.ietf.org/html/rfc3986), which version of the HTTP Protocol it uses etc.

    The application layer writes the data to the stream and passes it to the **transport** layer.

2. **The transport layer (TCP)**: Break the data into smaller parts and identifies them

    The **transport** layer reads the data from the stream, breaks the request in smaller chunks (TCP segments) and give each one a number and a checksum (an unique identifier that is checked to see if the data was corrupted or not).

    While researching about the TCP/IP model, I have seen articles saying that the **transport** layer turns data into packets (IP datagrams) and that the **internet** layer only sends them out. Other articles state that the **transport** layer breaks the data into TCP segments, that then are turned into even smaller chunks (packets) in the **internet** layer. I decided to use the second explanation, because that is how it is described in [Java Network Programming](http://shop.oreilly.com/product/9780596007218.do).

3. **The internet layer (IP)**: Create and dispatch packets

    The **IP** layer gets those chunks of data (TCP segments), breaks them into packets (IP datagrams) and passes them to the next layer. This is where data is addressed and routed.

    An IP datagram holds the IP addresses of the sender and the receiver, the payload data, its size, and other information. There are two versions of how this information is organized: [IPv4 and IPv6](https://www.arin.net/knowledge/ipv4_ipv6.pdf), the difference being the byte size of the IP addresses (32 vs 128 bytes). The [IPv4 was exploited](https://twitter.com/eastdakota/status/783841607963381760) recently in an attack that put down popular websites around the world.

4. **The host-to-network layer**: The *internet*

    Here is where the data *travels* from one device to another, the physical transportation of the data.

5. **The internet layer**: Receive packets

    This time, the **internet** layer reassembles the packets if they eventually got fragmented and make some checks to see if the they were corrupted. It then passes the packets to the next layer.

6. **The transport layer**: Reassembles packets

    The server **transport** layer now checks to make sure that all packets have arrived and are not corrupted. If something is missing or corrupted, it sends back a request for the missing packet. This is a characteristic of the TCP protocol. The [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol) protocol does not guarantee data integrity (it is used to transmit videos, for instance). When everything is right, it reassembles the data and writes it to a stream sent to the server.

7. **The application layer**: The server

    The server does its work (it also handles the HTTP protocol), finding the right data to be sent back. The response goes the other way around, through the same layers, back to the client.

The TCP/IP is not the only possible layer model and could be broken down into even smaller steps, specially the **host-to-network** layer. But this gives us an idea of how things work. Here is how it could be represented:

![Internet Protocol Suite](/images/tcp-ip.png)
