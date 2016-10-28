---
layout: post
title:  "The Problem with var Inside Objects"
date:   2016-10-27 14:21:32 -0500
category: apprenticeship
tags: [scala]
---

One of the first things we learn when we start writing Scala code is that `var` is mutable and `val`, immutable. And that we can use them whichever way we need. But no beginners tutorial says that we should not use `var`s inside an `object`. <!--more-->

It does not mean that it is a forbidden operation, only that it will give us a big headache when we try to understand why a value is not what we expected.

## What is an `object`

When we create an object using `object Foo { //... }`, we create a singleton &mdash; an instance of a class that can be instantiated only once. If we use Scala as a functional language, an object works like a module, or a namespace. So, we can do things like `Foo.someFunction(args)` to execute a function.

If we explicitly define a `class Foo { //... }` and then an object with the same name in the same source file, that object will be a *companion object* (a class and its companion object share each other private variables and methods).

In his book [Programming in Scala](http://www.artima.com/shop/programming_in_scala_3ed), Prof. Martin Odersky says that <sup>[1](#footnote1)</sup>:

> If you are a Java programmer, one way to think of singleton objects is as the home for any static methods you might have written in Java.

That sentence is the key to understand why `var`s are dangerous inside an object. A `static` variable is a class variable and it's shared with all instances of that class. If a `static` variable gets changed at any point, all instances that use that `static` variable will have the value updated. This means that, in Scala, it is almost like having a **global and mutable** variable.

## A `var` inside an object

Having a global mutable variable is possible, and that is why we can declare them inside an object without getting any compile error. But if we use them, we can end up having to deal with unexpected values. I found out about the problem when my unit tests started to randomly fail. It was hard to isolate the problem, because the failures would happen in different moments and return different errors.

This kind of problem happens because all the tests run against an object that don't get instantiated for each test block, so the `var`s can have different values depending on when/how the tests are executed.

## The solution

There are some hacks to fix the problem with the unit tests, like creating some kind of `reset` method. Most of the suggestions I saw while researching about it seemed to make the design more complicated than cleaner. My solution was to simply turn the object (`Negamax`) into a class, and then instantiate the object when needed.  

---
<p id="footnote1"> The book continues: <em>"A singleton object is more than a holder of static methods, however. It is a
first-class object."</em> </p>
