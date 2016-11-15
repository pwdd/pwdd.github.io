---
layout: post
title:  "Ports and Sockets"
date:   2016-11-14 20:55:32 -0500
category: apprenticeship
tags: [java]
---

The server has a socket that listens for connections at a port of a given number. When that socket is created and bound to a port, it starts accepting connections. It does it by creating yet another socket, that will process requests and responses. Wait. What?<!--more-->

There are two ways to deal with an explanation like the one above: just learn it *as is* and move forward; or try to understand all of it &mdash; sockets, listenings, ports, etc.

There is nothing wrong with the first option, if we are good with abstractions. [*A rose is a rose is a rose*](http://writing.upenn.edu/library/Stein-Gertrude_Rose-is-a-rose.html). A thing is what it is. (And I've never imagined that I would quote Gertrude Stein to talk about servers...). I usually lean toward the second way &mdash; which can easily lead one down to a rabbit hole, but that is another story.

Back to initial explanation, part by part:

## Sockets

In plain English, a socket is a place where something will be plugged in. This is helpful to understand what is a socket in network programming.

It is almost like a "place" (metaphorically speaking) where the client gets connected to the server and vice-versa.


```
|--------|----------|  connection  |----------|--------|
| client | [socket] |<------------>| [socket] | server |
|--------|----------|              |----------|--------|
```

Sockets, usually defined as *endpoints of the communication*, work similarly to an interface between the connection and the server/client.

## Ports

We have our computer on, and we have an email client, a FTP client and a browser open. When an email arrives, it should not be directed to the FTP client.

That's why ports exist: they help make the data get to the right place.

When a client sends a request, it sends it to a specific port. But how does a client know which port to use?

There are many [ports that are assigned](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt) to specific services. A HTTP connection happens at port 80, a FTP connection, at port 21. And there are thousands of unassigned ports.

When we start a server in out local machine, we define on which port it will be listening for connections. So when we enter `"localhost:<number>"` in the browser to send a request to the local server, we are saying that the request should be sent to a port with that given number.

## Sockets and ports

Sockets listen (wait for connection) on given ports. When we create a socket, we do it by binding the socket to a port. So when data is sent to a given port, it is processed by the socket associated with that port.

In Java, this is how we instantiate a server socket:

```java
new ServerSocket(portNumber);
```  

## Sockets in a connection

From the server perspective, we need 2 sockets. One that will be listening for connections and other one where the communication will happen.

If we successfully create a Server Socket and bind it to a port, we need to start accepting data. We do it by creating another socket, that will use the same port. This socket is where we are going to read from the `InputStream` (the client request) and write to the `OutputStream` (send response).

```java
try (
  // create a server socket to listen at a given port
  ServerSocket serverSocket = new ServerSocket(portNumber);

  // when server socket is bound to a port, start accepting connections
  // this creates a new socket
  Socket socket = serverSocket.accept();

  // send response through the socket
  PrintWriter out =
      new PrintWriter(socket.getOutputStream());

  // get request from the socket
  BufferedReader in = new BufferedReader(
      new InputStreamReader(socket.getInputStream()));
) {

  // do something with the data

} catch (Exception e) {
  System.out.println(e.getMessage());
}
```

And there we have it: "The server has a socket that listens for connections at a port of a given number. When that socket is created and bound to a port, it starts accepting connections. It does it by creating yet another socket, that will process requests and responses."
