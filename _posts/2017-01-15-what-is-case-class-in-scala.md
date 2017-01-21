---
layout: post
title:  "What is a case class in Scala?"
date:   2017-01-15 14:50:35 -0500
category: apprenticeship
tags: [scala]
---

A quick answer to the question of the title of this post is: it allows us to use pattern matching when comparing its instances and it does not need the keyword `new` in order to initialize an instance. But why? How?<!--more-->

## Defining a case class

One situation (not the *only* one) when case classes are useful is when we have multiple classes that extend a trait or an abstract class.

```scala
abstract class Player
case class Computer(mark: Symbol) extends Player
case class Human(mark: Symbol, messages: String) extends Player
case object SuperEspecialPlayer extends Player
```

When a case class is defined, the compiler will automatically create its companion object with a factory method, that will initialize instances of that class directly. That is the reason why we do not need the `new` keyword.

```scala
// (part of the) implicit companion object created by the compiler
// the object has other methods besides `apply`
object Computer {
  def apply(mark: String) = new Computer(mark)
}
```

By the way, a **companion object** is an object has exactly the same name and is defined in the same file that a class. It shares methods and even private fields with the parent class. And there is no need to worry about it when writing case classes. It is a compiler's job to do it.

## Accessing `val`s

We can write a "regular" class like this:

```scala
class Foo(_number: Int) {
  val number = _number
}

val f = new Foo(42)
f.number
// 42
```

Or using the easier way:

```scala
class Foo(val number: Int)

val f = new Foo(42)
f.number
// 42
```

With a case class, the argument is automatically defined as public `val`:

```scala
// case class Computer(mark: Symbol) extends Player
val computerPlayer = Computer('x)
computerPlayer.mark
//-> 'x
```

Because it is defined as a `val`, we cannot reassign it:

```scala
computerPlayer.mark = 'o
// error: reassignment to val
```

We can explicitly define the argument as a `var`, but that is not exactly a good idea. A case class is an **immutable class** (like `List`, for instance, that is also an immutable class). We can, however, make copies &mdash; which creates new objects.

```scala
val computer = Computer('x)
val anotherComputer = computer.copy(mark = 'o)
anotherComputer.mark
//-> 'o
```

## Extending a case class with constructor

Because the arguments passed to a case class constructor are `val`s, they can cause a conflict between the extended class and the case class.

In a "regular" class, if we do:

```scala
class Foo(number: Int) {
  val number = number
  // error: reassignment to val
}
```

What happens in here is that `number`, in the constructor, is a `val`, but it is not a **member of the class Foo**. That is why we cannot do

```scala
class Foo(number: Int)
val f = new Foo(2)
f.number
// error: value 'number' is not a member of Foo
```

Understanding this is helpful when we try to understand how to understand a class that has a constructor. For example:

```scala
// THIS WILL NOT WORK
abstract class Player(_mark: Symbol) {
  val mark = _mark
}

case class Computer(mark: Symbol) extends Player(mark)
// error: overriding value 'mark' in class Player of type Symbol;
// value 'mark' needs `override' modifier
```

This happens because `mark` is a member of the class `Player` and, `Computer`, by extending `Player`, has access to it. When we say `case class Computer(mark: Symbol)` we are saying that `mark` is a member of `Computer`, and it should explicitly override the member of `Player`.

It is just a matter of changing the name of the constructor, nothing more.

```scala
abstract class Player(_mark: Symbol) {
  val mark = _mark
}

case class Computer(renamedMark: Symbol) extends Player(renamedMark)

val comp = Computer('o)
com.mark
//-> 'o
```

## Pattern Matching

One of the reasons why using case classes when they extend a trait or abstract class is that, when we define a case class, the compiler also automatically generates an `equals` method. This allows us to compare objects by structure (instead of by reference).

```scala
val c1 = Computer('x)
val c2 = Computer('x)

c1 == c2
// true
```

This means that we can use pattern matching when dealing with case classes.

```scala
def currentPlayerMessage(player: Player): String = player match {
  case Computer(mark) => "Computer is playing with " + mark
  case Human(mark, _) => "Human is playing with " + mark
  case _ => "Game over"
}
```
