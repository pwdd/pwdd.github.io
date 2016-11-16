---
layout: post
title:  "Testing a socket input and output"
date:   2016-11-15 8:55:32 -0500
category: apprenticeship
tags: [java]
---

Understanding how to develop a server that is able to receive a simple GET request and send a simple response is relatively easy. There are tons of good resources to help us with that. Testing the server is the most difficult part.<!--more-->

It took me only a few days to learn about requests, responses, HTTP protocol, sockets and everything necessary to write a basic server &mdash; that sends back "hello, world" to every GET request. However, I got stuck for a couple of weeks to figure out a way to test it. The original idea was to put something together, see how it worked, and start over, using TDD.

For most part, the plan went well. There is nothing complicated about testing if the response body has a given string. If we followed the [Single Responsibility Principle](https://8thlight.com/blog/elizabeth-engelman/2015/01/22/single-responsibility-principle-why-does-it-matter.html), we probably have a method whose only purpose in life is to return a string accordingly to the argument passed to it. Hopefully, the argument is only the strictly necessary for the method to know which string to return. The test would just ensure that the returned string is the right one. It could be something like this:

```java
// method to be tested
String responseBodyForRequested(String uri) {
    if (uri.equalsIgnoreCase("/hello")) {
      return "Hello, world";
    } else {
      return "something else";
    }
  }

// tests
@Test
public void responseBodyForHello() {
  assertEquals("Hello, world", responseBodyForRequested("/hello"));
}

@Test
public void responseBodyForAnythingElse() {
  assertEquals("something else", responseBodyForRequested("/"));
}
```

As most of the work is about manipulating strings, the tests are simple to put together. But, how to test that a server socket accepts connections? That it sends responses? That it receive requests? And how to do it without having to couple the server socket to an specific socket that will process input and output?

## Mocking a socket

Probably one of the things that helped me the most was an advice from one of my mentors, who said that "*If a class is difficult to test, we know that our tests are trying to tell us that it is time to refactor*". From there, I worked on separating everything so that I could test things as isolated as possible &mdash; a method to get the input, one to send the output, one to bind a server socket to a port, one to create the socket for input and output and so on.

*(From now on, I will refer to the socket that gets bind to a port and `serverSocket`, the socket that input and output pass through as `socket` and, the client socket as `clientSocket`)*

### The first test:

- **A connection only happens if a `serverSocket` is listening at a port**.

In this case, we only need to make sure that the `serverSocket` is listening for connections, and nothing else. We test is by creating a `clientSocket` and trying to connect. There is no need to worry about requests and responses.

```java
@Test
  public void serverAcceptsConnection() throws IOException {
    startServer(); // creates the `serverSocket`

    // create a `clientSocket` that will try to connect to a serverSocket
    // that has the hostname 'localhost'
    // and listens at port number 8080
    try(Socket ableToConnect = new Socket("localhost", 8080)) {
      assertTrue("Accepts connection when server in listening",
                 ableToConnect.isConnected());
      // close the `clientSocket`
      ableToConnect.close();
    } catch (Exception e) {
      System.out.println(e.getMessage());
    }
    // close the 'serverSocket'
    server.stop();

    try {
      // now that `serverSocket` is closed
      // try to connect another `clientSocket` to the same `serverSocket`
      new Socket("localhost", 8080);
      fail("Cannot connect if server socket is not listening");
    } catch (Exception e) {
      // assert that the exception is thrown and is the right exception
      assertEquals("Connection refused", e.getMessage().trim());
    }
  }
```

## Second test:

- **make sure the server can receive a request**

For this test, we need to mock a socket. The idea is that the `serverSocket` will get an input from a `mockSocket` and send an output through the `mockSocket`. After all, we are testing the `serverSocket`, we can use a mock instead of a `socket`. But how do we go about mocking a socket?

Everything we need from a socket are the methods `getInputStream` and `getOutputStream`. For this second test, we actually need only `getInputStream`.

```java
public class MockSocket extends Socket {
  // a mockSocket constructor does not need hostname and port number,
  public MockSocket() {}

  // return a InputStream with a dummy request
  public InputStream getInputStream() {
    return new ByteArrayInputStream("GET / HTTP/1.1\nHost: localhost".getBytes());
  }

  // coming up next! :)
  public OutputStream getOutputStream() {
    return new OutputStream() {
      @Override
      public void write(int b) throws IOException {

      }
    };
  }
}
```

This is the method we are testing:

```java
// get the InputStream from the `socket` and turn it into a BufferedReader
BufferedReader getRequestFrom(Socket socket) throws IOException {
  return new BufferedReader(new InputStreamReader(socket.getInputStream()));
}
```

And here is the test:

```java
@Test
  public void serverAcceptsRequest() throws IOException {
    startServer();
    MockSocket mockSocket = new MockSocket();
    String request = server.getRequestFrom(mockSocket).readLine();
    assertEquals("GET / HTTP/1.1", request);
  }
```

There is nothing special about that test. It makes the `serverSocket` start listening to a port, and call `getRequestFrom` using a `mockSocket` as argument. Then we check if the returned value of `getRequestFrom` corresponds to the InputStream from the `mockSocket`.

## Third test:

- **make sure that the server is able to send responses**

So far, this had been the trickiest part, even with a mock in place. Here is the method under test:

```java
void sendResponse(Socket socket, String responseBody) throws IOException {
  PrintWriter out = new PrintWriter(socket.getOutputStream());
  out.print(responseHeader);
  out.print(responseBody);
  out.flush();
}
```

The `PrintWriter` is just used to write the response. How to check that the server is actually sending some data to some client? Would it be necessary to create yet another mock, representing the client socket, to check if it receives the right response? This option would only add complexity to the test, and it would be hard to guarantee that `serverSocket` works by sending something, because we could be testing if a client socket *receives* something.

The solution is again on the `mockSocket`. When we call `print` on `out`, we are calling the method `write` from the OutputStream. The method `sendResponse` does not depend on a client Socket to exist, so it can be tested without it. Here is how the final `MockSocket` class could be:

```java
public class MockSocket extends Socket {
  // create an empty list of bytes
  private List<Byte> bytesList = new ArrayList<>();

  public MockSocket() {}

  public InputStream getInputStream() {
    return new ByteArrayInputStream("GET / HTTP/1.1\nHost: localhost".getBytes());
  }

  public OutputStream getOutputStream() {
    return new OutputStream() {
      @Override
      // every time we call `write` (out.print),
      // we add the bytes to the list 'bytesList'
      public void write(int b) throws IOException {
        bytesList.add((byte) b);
      }
    };
  }

  // this method does not exist in the super class 'Socket'
  // it is used to return the string formed by the bytes added to 'bytesList'
  public String output() {
    byte[] converted = toByteArray(bytesList);
    return new String(converted, StandardCharsets.UTF_8);
  }

  // convert a list of Bytes objects to an byte array
  private byte[] toByteArray(List<Byte> byteList) {
    byte[] byteArray = new byte[byteList.size()];
    int index = 0;
    for (byte b : byteList) {
      byteArray[index++] = b;
    }
    return byteArray;
  }
}
```

Creating the mock is the most complicated part. After that, the test is simple: it sends a response (independent from `socket` and client socket). We just check that `write` was called and all those bytes that we saved in the `bytesList` correspond to the string we sent as response.

```java
@Test
  public void serverSendsResponse() throws  IOException {
    startServer();
    MockSocket mockSocket = new MockSocket();
    server.sendResponse(mockSocket, "foo");
    String body = mockSocket.output();
    assertEquals("foo", body);
  }
```
