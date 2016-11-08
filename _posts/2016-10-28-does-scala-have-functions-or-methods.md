---
layout: post
title:  "Does Scala have functions or methods?"
date:   2016-10-28 10:21:32 -0500
category: apprenticeship
tags: [scala]
---

If a function is declared inside an object, is it a function or a method? <!--more-->

At first, it seems easy to distinguish between then:

```scala
def aMethod(x: Int) = x + 1 // aMethod: (x: Int)Int

val aFunction = (x: Int) => x + 1 // aFunction: Int => Int = <function1>

aMethod
//-> error: missing argument list for method aMethod

aFunction
//->  Int => Int = <function1>
```

In the examples above, `aMethod` is a method &mdash; the error message clearly has the word *method*. And `aFunction` is a function, because the returned value shows `<function1>`.

But it is not that simple.

On his book [Programming in Scala](http://www.artima.com/shop/programming_in_scala_3ed), Prof. Martin Odersky says that  

> Function definitions start with `def`

This means that our first *method* (`def aMethod(x: Int) = x + 1`) is actually a function. In another part of the book, one of the code snippets used in is this one:

```scala
val nameHasUpperCase = name.exists(_.isUpper)
```

Followed by the explanation:

> The predicate `_.isUpper` is an example of a **function literal** in Scala.
It describes a function that takes a character argument (represented by the
underscore character), and tests whether it is an upper case letter.

[`isUpper`](http://www.scala-lang.org/api/current/scala/runtime/RichChar.html#isUpper:Boolean) is a method inside `RichChar` class.

In another book, [Programming Scala](http://shop.oreilly.com/product/0636920033073.do) (yes, the names are almost the same), the authors Dean Wampler and Alex Payne explain that, when declared inside an object, a *method* is **lifted** to a function if it does not modify characteristics of the object.

The `lift` method will probably be subject of another post, because it is able to do lots of interesting things. In the case of *lifting a method to be a function*, it means that it will turn the method into a *function object*.

## Function object

A function is an object that inherits from different traits. A function that accepts one parameter inherits from the trait `Function1`. This means that we could declare a function by creating an object:

```scala
// `addOne` extends the trait `Function1`, that takes an `Int` and returns an `Int`.
object addOne extends Function1[Int, Int] {
  def apply(x: Int): Int = x + 1
}

addOne(1)
//-> Int = 2
```

## Turning methods into functions

If the compiler can *lift* a method to be a function, we can also do it.

```scala
object Foo {
  def addOne(x: Int) = x + 1
}

Foo.addOne(1)
//-> Int = 2

val liftedAddOne = Foo.addOne _
//-> liftedAddOne: Int => Int = <function1>

liftedAddOne(1)
//-> Int = 2
```

If we had set `liftedAddOne` to be `Foo.addOne` or `Foo.addOne()` it would throw an error, complaining about the missing argument. This is fixed by adding the `_` (underscore) after the method, so it is not called and it is, instead, lifted to be a function object.

After learning all this, it seems that, most of the time, the difference between functions and methods it is just a matter of terminology, and not an important distinction. Basically, what makes a function a function is how we use it.
