---
layout: post
title:  "Polymorphism in Scala"
date:   2016-12-16 14:50:35 -0500
category: apprenticeship
tags: [scala]
---

Probably one of the funnest part of learning a language is to find out how the given language deals with polymorphism. Scala, as a functional and an object-oriented language, can implement both subtype (usually associate with OO languages) and generics (with functional ones) polymorphism.<!--more-->

First things first, polymorphism is when a function can handle arguments of many types. For instance, some languages make it possible to write things like `"a" + "b" == "ab"` and `1 + 2 == 3`. The `+` function can operate on different type of arguments. Another place where we can identify polymorphism is when a type can have instance of many types. It sounds weird, but it is actually kind simple. For example, in Java we could do:

```java
abstract class Animal {
  // some methods
}

class Dog extends Animal {
  // the implementation
}

class Cat extends Animal {
  // the implementation
}
```

In the above example, a type `Animal` can have instances of two types: a `new Dog()` of type `Dog` and a `new Cat()` of type `Cat`.

This example, of extending a class, is what is called *subtyping*. The first example is called *generic* polymorphism. In Scala, we have both. A very simple example is the implementation of a [immutable linked-list](http://cslibrary.stanford.edu/103/LinkedListBasics.pdf) (the example is from the online [course](https://www.coursera.org/learn/progfun1) given by Prof. Martin Odersky, who created Scala)

```scala
trait List[T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
}
```

The `trait` in Scala is like an `interface` in Java. Both traits and classes can be parameterized &mdash; they can accept a type as argument &mdash;. The `List[T]` trait accepts a generic type `T`.

Then, the implementation:

```scala
class Nil[T] extends List[T] {
  def isEmpty: Boolean = true
  def head: T = throw new NoSuchElementException
  def tail: List[T] = throw new NoSuchElementException
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  def isEmpty: Boolean = false
}
```

The example shows subtyping (instances of `List[T]` can be `Cons[T]` or `Nil[T]`) and generics (a `List` can handle arguments of any type). A method to create a list could be written like this:

```scala
def singleton[T](element: T) = new Cons(element, new Nil)
```

Calling that method could be done like this:

```scala
singleton(1)
// Cons[Int] = Cons(1, Nil)

singleton("a")
// Cons[String] = Cons("a", Nil)
```

It is not necessary to declare the type because the Scala compiler can infer it.

## Bounds and covariance

A bound is a limitation that we can impose on the parameterization. If we needed to filter a set of integers to return a new set of integers, we could say that: **1)** if the set is empty, returns an empty set; **2)** if the set is non-empty, returns a non-empty set or throws an exception.

The function to get only the positive numbers could be written like this:

```scala
def allPos(S <: IntSet)(arg: S): S = // implementation
```

The **bound** we defined `IntSet` is the upper bound for the types that `allPos` accepts. That is why that `<:` is there, to denote that `S` is a subtype of `IntSet`. It is also possible to define a super bound. And, of course, we can do a lot more. For instance, we could say that a method accepts `S >: NonEmpty <: IntSet` &mdash; which means that it accepts a `S` that is a subtype of `IntSet` and a supertype of `NonEmpty`.

Now, what if we need use those types in a collection. We could say that `List[S] <: List[IntSet]` &mdash; a list of `S` is also a subtype of a list of `IntSet`. This would be a [**covariant**](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)) relationship.

## The Liskov Substitution Principle

The principle defines when a type can be a subtype. It says that if `A` is a subtype of `B`, it should be able to do everything `B` does. And it is possible to observe it comparing Java and Scala covariance.

```java
// In Java, arrays are covariant
NoEmpty[] a = new NonEmpty[] { new NonEmpty(1, Empty, Empty) };
IntSet[] b = a;
b[0] = Empty;
NonEmpty s = a[0];
```

The Java example above would throw an the `ArrayStoreException` at runtime, because we are setting an `Empty` value (`a[0]`) to a `NonEmpty` (`s`).

```scala
// In Scala, arrays are NOT covariant
val a: Array[NonEmpty] = Array(new NonEmpty( 1, Empty, Empty))
val b: Array[IntSet] = a
b(0) = Empty
val s: NonEmpty = a(0)
```

The Scala example would throw an exception at the compiler time, when we tried to define `b`, because `Array[NonEmpty]` is not a subtype of `Array[IntSet]`
