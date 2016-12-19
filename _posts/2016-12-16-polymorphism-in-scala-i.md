---
layout: post
title:  "Polymorphism in Scala - I"
date:   2016-12-16 14:50:35 -0500
category: apprenticeship
tags: [scala]
---

Scala, as a functional and an object-oriented language, can implement both subtype and generic polymorphism. The first one is usually associated with OO languages, and the second, with functional.<!--more-->

First things first, **polymorphism** is when a function can handle arguments of many types. For instance, some languages make it possible to write things like `"a" + "b" == "ab"` and `1 + 2 == 3`. The `+` function can operate on different types of data (strings and integers). Another place where we can identify polymorphism is when a type can have instances of many types. It sounds weird, but it is actually simple. For example, in Scala we could do:

```scala
abstract class Animal {
  // some methods
}

class Dog extends Animal {
  // implementation
}

class Cat extends Animal {
  // implementation
}
```

In the above example, a type `Animal` can have instances of two types: a `new Dog()` of type `Dog` and a `new Cat()` of type `Cat`.

This example, of extending a class, is what is called *subtyping*. The first example is called *generic* polymorphism. Both can be observed in the implementation of a [immutable linked-list](http://cslibrary.stanford.edu/103/LinkedListBasics.pdf).

```scala
trait List[T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
}
```

The `trait` in Scala can be seen as an `interface` in Java (although it is not always the case: a trait is compiled to a Java interface only if it has [abstract members](http://www.artima.com/pins1ed/abstract-members.html); otherwise, it has no correspondence in Java code).

Traits and classes can be parameterized &mdash; they can accept a type as argument. The `List[T]` trait accepts a generic type `T`.

Then, the implementation:

```scala
class Nil[T] extends List[T] {
  def isEmpty: Boolean = true
  def head: Nothing = throw new NoSuchElementException
  def tail: Nothing = throw new NoSuchElementException
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  def isEmpty: Boolean = false
}
```

The example shows subtyping (instances of `List[T]` can be of type `Cons` or `Nil`) and generics (a `List` can handle arguments of any type). A method to create a list could be written like this:

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

When calling `singleton`, we could pass in the type (`singleton[Int](1)`), but it is not necessary because the Scala compiler can infer it.

## Abstract class vs trait

Deciding between an abstract class or a trait is not difficult: use trait whenever possible.

Although an abstract class has more pros than traits, the recommendation &mdash; that comes from the book [Programming in Scala](http://www.artima.com/pins1ed/traits.html#12.7), by Prof. Martin Odersky &mdash; is to favor trait because, if necessary, it is easier to turn it into an abstract class later, and because it is possible that a subtype can extend multiple traits.

```scala
trait X {
  // ...
}

trait Y {
  // ...
}

class Foo extends X with Y {
  // ...
}
```

The situations in which an abstract class would be preferred would be when:

- Performance is important. The invocation of a class is faster at runtime.
- We need constructor parameters.
- A Java class will inherit from it.

## Inheritance and composition

This is not a post about design decisions or the benefits of [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance), but how Scala deals with polymorphism. And both composition and inheritance are ways to implement polymorphism &mdash; a way to define a class in terms of another. Inheritance is what we've seen so far, when a class *extends* another. Composition is when a class holds a reference of another. And, of course, we can also use composition when programming in Scala.

```scala
abstract class Form {
  // ...
}

class Line extends Form {
  // ...
}

// 'Square' inherits from 'Form'
// 'Square' composes with 'Line'
class Square extends Form {
  val line = new Line
}
```

- Second part: [*Polymorphism in Scala &mdash; II*]({% post_url 2016-12-17-polymorphism-in-scala-ii %})
