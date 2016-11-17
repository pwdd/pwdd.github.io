---
layout: post
title:  "A multithreaded server"
date:   2016-11-16 8:55:32 -0500
category: apprenticeship
tags: [java]
---

The use of packets in the TCP/IP connection made possible for multiple transfers of data to happen at the same time. A packet is very small and so many of them can be transferred on the same wire. And, of course, servers can accept more than one connection at a time. This is often done with the use of `Threads`.<!--more-->

A thread happens inside a process. They are both *sequences of execution*, but a **process** provides all the resources needed to execute a program, and those resources are exclusive to that program. Each process starts with a single thread. For instance, when we open a text editor, we start a new process.

A **thread** is an entity within a process. If a new thread is added to a process, it will share the memory and system resources used inside the process with other threads. Using the text editor example, let's say that it automatically saves what we write while we are still editing. This usually means that saving and editing are handled in different threads inside the same process.

And what it has to do with a server?

If we want our server to accept more than one connection at a time, we need it to be able to start a new thread to handle requests and responses. In Java, there are two ways to start using thread: extending the class `Thread` or implementing the interface `Runnable`. According to [Java documentation](http://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html), `Runnable` is preferred when the only thing we need is to override the `run()` method &mdash; and that's the case.

## What needs to use a new thread?

The server uses two sockets: the server socket that listens at a port, and a socket that handles the communication with the client. The second one can make use of threads to handle multiple connections.

```java
class ConnectionHandler implements Runnable {
  private Socket socket;

  ConnectionHandler(Socket socket) {
    this.socket = socket;
  }

  BufferedReader getRequestFrom(Socket socket) throws IOException {
    return new BufferedReader(new InputStreamReader(socket.getInputStream()));
  }

  void sendResponse(Socket socket, String response) throws IOException {
    PrintWriter out = new PrintWriter(socket.getOutputStream());
    out.print(response);
    out.flush();
  }

  @Override
  public void run() {
    try {
      getRequestFrom(socket);
      sendResponse(socket, "some response");
      socket.close();
    } catch (Exception e) {
      System.out.println(e.getMessage());
    }
  }
}
```

`run()` is equivalent to `main()` in a single-threaded program. When `run()` ends, the thread dies.

After implementing `Runnable`, we just need to pass the object to a new thread. The `start()` method will call `run()` from the object and internally execute it.

```java
public static void main(String args[]) {
  ServerSocket serverSocket = new ServerSocket(portNumber);

  while (true) {
    try {
      Socket socket = serverSocket.accept();
      new Thread(new ConnectionHandler(socket)).start();
    } catch (Exception e) {
      System.out.println(e.getMessage());
      return;
    }
  }
}
```
