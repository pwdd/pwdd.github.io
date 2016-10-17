---
layout: post
title:  "Basic Scala Syntax"
date:   2016-10-14 18:57:32 -0500
category: apprenticeship
tags: [scala]
---

The fact that Scala can be written as an Object Oriented or a Functional language gives us at least two different ways to accomplish the same thing. But it is not only that: common tasks, like defining a function/method, also can be done in different ways. <!--more-->

## Defining a variable

Scala differentiates mutable from immutable variables.

```scala
// value is immutable
val x = 1

// variable is, well, mutable
var y = 1
```

This means that when we define a `val` inside an object, we will only have a *getter* for that value, while if we define a `var`, we will get also a *setter*.

```scala
object Foo {
  val x = 1
  var y = 1
}

Foo.x //-> Int = 1
Foo.x = 2
// error: reassignment to val

Foo.y //-> Int = 1
Foo.y = 2
Foo.y //-> Int = 2
```

The mutability is relative to the definition of a value/variable. This means that `x` will not have other value associated with it other than 1, while `y` can have another value associated with it. This means that a variable (mutable) could be associated with an immutable data type:

```scala
object Bar {
  var a = List(1, 2, 3)
}

Bar.a //-> List[Int] = List(1, 2, 3)
Bar.a = List(2, 4, 6)
Bar.a //-> List[Int] = List(2, 4, 6)
```

In this case, `a` is mutable, but `List` is not.

One last thing: a `val` is not a constant, that is written with capital letter.

## Type inference

Scala is statically typed but, as we've seen so far, sometimes it's not necessary to write the type of an expression. The type will be inferred by the compiler:

```scala
object Bar {
  var a = List(1, 2, 3)
}

Bar.a //-> List[Int] = List(1, 2, 3)

Bar.a = Array(2, 4, 6)
//error: type mismatch;
// found   : Array[Int]
// required: List[Int]
```

Unless it is something as simple as defining a value, it's good habit to write the type, because it is a good documentation for our program.

## What is that `object` thing?

An `object` is an instance of a class, but it can also be used as a *module* when we are using Scala as a functional language, as I did in the examples above. Objects declared like that are *singletons* &mdash; they are instances of a class that can only be instantiated once.

```scala
// a 'module' or a 'singleton' object

object Baz {
  val z = 0
}

new Baz
// error: not found: type Baz

Baz.z
//-> Int = 0

Baz.getClass
//-> Class[_ <: Baz.type] = class Baz$
```

A regular object is defined just like in any other OO language.

```scala
class FooBar {
  val x = 0
}

val obj = new Foobar
// obj: FooBar = FooBar@152d5750

obj.x
//-> Int = 0
```

## Class constructor

When defining a class, there is no definition of a constructor, like the `initialize` in Ruby, for example. This is what we would do if we needed to pass parameters to the constructor when instantiating an object.

```scala
class Person(first: String, last: String, yearsOld: Int) {
  val firstName = first
  val lastName = last
  var age = yearsOld
}

val aPerson = new Person("John", "Smith", 20)

aPerson.firstName //-> John
aPerson.age //-> 20
```

Note that the name of the parameters must be different from the name of the instance variables. This is because the parameters are themselves immutable values, but not as fields of an object.

The example above could be shorter if written like this:

```scala
class Person(val firstName: String, val lastName: String, var age: Int)

val aPerson = new Person("John", "Smith", 20)

aPerson.firstName //-> John
aPerson.age //-> 20
```

Another way to do it would be using a `case class` that automatically defines the *getters*. In this example, we would have to explicitly use `var` for `age` field. Otherwise, it would be a `val`.

```scala
case class Person(firstName: String, lastName: String, var age: Int)

val aPerson = Person("John", "Smith", 20)

aPerson.firstName //-> John
aPerson.age //-> 20
```

**Case classes** are just like regular classes, but they:

1. Automatically define *getters*

2. Do not require the constructor `new` when creating an object (`val aPerson = Person("John", "Smith", 20)`)

3. Are useful when using pattern matching to decompose data structure. And this is a much bigger topic that will probably become a post itself. 

## Defining a function

Functions also use the type inference and will not fail if we forget to give it a return type. The exception is when we write recursive functions, when the type declaration is mandatory.

```scala
def sum(a: Int, b: Int) = a + b

sum(1, 2) //-> 3
```

When the function body can be written in just one line, like the example above, the `=` sign is enough for the definition. When it is longer, it's necessary to use curly braces

```scala
def printNumber(number: Int): Unit = {
  if (number == 0) println("The number is 0")
  else if (number < 0) println("The number is negative")
  else println(number)
}
```

As we can see, although Scala is statically typed, there is no need to turn `number` into a string in order to pass it to `println`. The string interpolation for this simple expression will automatically change `Int` into `String`.

The return type of the above function is `Unit`, which means that the function returns no value (`println` returns no value). Note that the function does return, so `Unit` is the right type. When I function does not return, the type would be `Nothing`.
