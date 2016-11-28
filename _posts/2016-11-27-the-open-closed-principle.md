---
layout: post
title:  "The Open-Closed Principle"
date:   2016-11-27 15:50:35 -0500
category: apprenticeship
tags: [design-principles]
---

From all the [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles, I think that the "O" is the most difficult to understand. The definition of the Open-Closed Principle (OCP) says that *Software entities (classes, modules, functions etc) should be open for extension, but closed for modification*. We use the design principles in order to be able to adapt for future changes without having to modify existing code, so that definition seems redundant and unhelpful. <!--more-->

But if we think only about the *software entities*, it makes more sense. The goal of using the design principles is to be able to extend our program without too much hassle, and in order to accomplish that, we need to make sure that each function, each class, each package can also be extended without problem.

It might seem nonsensical the idea of extending a class without changing the existing code. What it means is that we can implement changes by *adding* code, instead of *changing* the existing one.

While developing the HTTP server, I had a class named `Responder` (responsible for building up the response to the client) depending on a `FileHandler` (that builds up a HTML index page).

I had something similar to this:

```
|-----------|                 |-------------|
| Responder | <-------------- | FileHandler |
|-----------|                 |-------------|
```

```java
// violating OCP
class Responder {
  private FileHandler fileHandler;

  Responder(String directoryName) {
    this.fileHandler = new FileHandler(directoryName);
  }

  String bodyForRequested(String uri) {
    if (uri.equalsIgnoreCase("/hello")) {
      return "Hello, world";
    } else {
      return fileHandler.index();
    }
  }
}
```

`Responder`, the way it is defined above, is violating OCP. If I need to handle the request for another URI, I have to *change the code*, by adding a condition to `bodyForRequested()`;

The way to solve the problem is to follow OCP and create an interface between `FileHandler` and `Responder`.

```
                                                     |-------------|
                                                 ___ | FileHandler |
|-----------|              |----------|     ___/     |-------------|
| Responder | <----------- | IHandler | ___/
|-----------|              |----------|    \___     |-------------------|
                                               \___ | HelloWorldHandler |
                                                    | -------------------|
```

```java
interface IHandler {
  boolean canRespond(String uri);

  String respond(String uri);
}

class FileHandler implements IHandler {
  // implementation
}

class HelloWorldHandler implements IHandler {
  // implementation
}

class Responder {
  private final IHandler[] handlers;

  Responder(IHandler[] handlers) {
    this.handlers = handlers;
  }

  String bodyForRequested(String uri) {
    for (IHandler handler : this.handlers) {
      if (handler.canRespond(uri)) {
        return handler.respond(uri);
      }
    }
    return "";
  }
}
```

Now, `Responder` depends on something abstract and less likely to change (the `IHandler` interface). And, if I need to handle another URI, I just create a new class that implements `IHandler`, without having to modify `FileHandler` or `Responder`.

## When to use OCP

A violation of OCP is usually easy to identify: if a class depend on another concrete class, there it is the violation. Conditional statements depending on the *type* of something are also a sign that the OCP is not being followed. But it is not always easy to create abstractions to solve the problem. In [*Agile Software Development &mdash; Principles, Patterns, and Practices*](https://www.pearsonhighered.com/program/Martin-Agile-Software-Development-Principles-Patterns-and-Practices/PGM272869.html), Robert C. Martin points out that

> (...) conforming to OCP is expensive. It takes development time and effort to create the appropriate abstractions. Those abstractions also increase the complexity of the software design. (...) Clearly, we want to limit the application of the OCP to changes that are likely.

> How do we know which changes are likely? We do the appropriate research, we ask the appropriate questions, and we use our experience and common sense. And after all that, *we wait until the changes happen!*

What that means is that we should let the change happen and then create an abstraction that would protect the entity against changes of that kind. Using the `Responder` example, it is protected against the need to implement new handlers, and only that &mdash; for now.

Let's say that we are developing an application to be used in the command line. This application has a `Printer` class that prints out strings to the console. We know that we could be asked to change the app so that it can be used in web browsers. We could create an abstraction `Printer` that would be implemented by `ConsolePrinter`. And, in the future, we would be able to easily add a `BrowserPrinter`. But, unless we know for some reason that it is a real possibility that the change is going to happen, we should not have the abstraction. The requirement might never happen and, instead of making things cleaner, we would end up creating unnecessary complexity.
