---
layout: post
title:  "Java IO Streams"
date:   2016-11-07 09:55:32 -0500
category: apprenticeship
tags: [java]
---

In the Internet Protocol Suite (TCP/IP &mdash; *subject of the previous post*), the application layer on the client side writes to a *stream* that is then passed to the transport layer. On the server side, the layer reads from a *stream* after data is reassembled by the transport layer. But what is that *stream* thing?<!--more-->

When talking about servers, the streams are `.java.io.InputStream` and `java.io.OutputStream`. They are a continuous flow of data, connected to some source. They write to or read from that source. Note that it is different from [`java.util.Stream`](http://www.oracle.com/technetwork/articles/java/ma14-java-se-8-streams-2177646.html), an interface implemented in Java 8.

An IO Stream can be byte based (access a source byte by byte) or character based. When we see \*Stream, like `FileOutputStream` or `TelnetInputStream`, they are usually based on bytes. Those that have `Reader` or `Writer` on their names are based on characters.

## `OutputStream`

The `OutputStream` and its subclasses &mdash; `FileOutputStream`, `ByteArrayOutputStream` and others &mdash; implement the methods `write()`, `flush()` and `close()`. The last two are used because an IO Stream waits for data to be read/written, so it is necessary to explicitly `flush` (otherwise, we could have a program *hanging* and never writing anything to the source) and `close` (to release the resources being used by the stream).

## `InputStream`

This class and its subclasses implement `read()`, `close()`, `skip()` and `available()`. `read()` returns an `int` containing the byte value of the byte read, and `-1` when there is nothing more to be read. The `available()` returns the number of bytes that can be read or `skip`ed.

## Buffering IO Streams

Because `InputStream` and `OutputStream` will read/write data byte by byte, it is sometimes necessary to optimize those actions. Here is where the buffer (a byte array named *buf*) comes in.

The `BufferedOutputStream` will wait till the `OutputStream` is flushed and write it all to a source.

The `BufferedInputStream` is used a bit differently: when `read()` is called, it tries to read data from a buffer. Only when all the data is read from it and the buffer is empty, `read()` is used to read from the source.

## Putting it all together

I have been working on writing a HTTP server in Java, using the `ServerSocket` and `Socket` classes. This means that I had to implement a way to read the request (the `InputStream`) and write the response (the `OutputStream`). And here is how it goes:

```java
// 'socket' is the socket used to talk to the client
// 'in' gets the InputStream that comes from that socket,
// filters it into a InputStreamReader (read the characters)
// that is then filtered into a 'BufferedReader',
// that works just like a 'BufferedInputStream'
// but it holds characters instead of bytes
BufferedReader in = new BufferedReader(
                      new InputStreamReader(socket.getInputStream()));

// 'PrintWriter' implements the methods 'print' and 'println'
// It is used to print the HTTP header and the payload to the OutputStream
// BufferedWriter does not implement those methods
PrintWriter out = new PrintWriter(
                    new BufferedWriter(
                      new OutputStreamWriter(socket.getOutputStream())));
```

After having those `in` and `out` in place, it is just a matter of organizing how the data passed back and forth between client and server is used.
