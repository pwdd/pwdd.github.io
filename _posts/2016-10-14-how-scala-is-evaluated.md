---
layout: post
title:  "How Scala Is Evaluated"
date:   2016-10-14 20:57:32 -0500
category: apprenticeship
tags: [scala]
---

The same way that expressions can be written in different ways, code written in Scala can also be evaluated in different ways. <!--more-->

## Call by value

By default, Scala evaluation is **call by value**, that follows the substitution model, similar to the evaluation of lambda calculus in mathematics. It reduces an expression to a value.

```scala
val x = 1
val y = 2

   (1 + x) * y
// (1 + 1) * y
//    2    * y
//    2    * 2
//-> 4
```

Functions are evaluated the same way. First, the parameters are evaluated; then the function name is replaced by it's body and, at the same time, the named parameters are replaced by its values.

```scala
def square(x: Int): Int = x * x

def sumOfSquares(x: Int, y: Int): Int = square(x) + square(y)

   sumOfSquares(3, 2 + 2)
// sumOfSquares(3, 4)
// square(3) + square(4)
// 3 * 3     + square(4)
//   9       + square(4)
//   9       + 4 * 4
//   9       + 16
//-> 25
```

The substitution model of evaluation can only be applied to functions that do not have side effects. When we increment the value of a variable, for example, it would be considered side effect, and it would be difficult to keep track of that given value. This would be one reason to avoid mutable variables.

# Call by name

The substitution model also cannot be used to evaluate expressions that cannot be reduced to a value, like an infinite loop. There is another way of doing it, the **call by name** evaluation, in which the expression is applied to unreduced arguments.

```scala
   sumOfSquares(3, 2 + 2)
// square(3) + square(2 + 2)
// 3 * 3     + square(2 + 2)
// 9         + square(2 + 2)
// 9         + (2 + 2) * (2 + 2)
// 9         + 4       * (2 + 2)
// 9         + 4       * 4
//-> 25
```

The **call by value** evaluates an expression only once, while **call by name** will not evaluate an argument if it is not used by the function body. There are situations in which one has more advantages than other. For instance:

```scala
def infiniteLoop(a: Int): Int = infiniteLoop

def id(x: Int, y: Int) = x

// call by name
id(1, infiniteLoop)
// `loop` does not matter for the final result, so it is not evaluated
//-> 1

// call by value
id(1, infiniteLoop)
// will never terminate because it is an infinite loop
```

## The default evaluation

In Scala, the method of evaluation is **call by value (CBV)**, because, although **cal by name (CBN)** terminates more often, **CBV** is more efficient in most cases.

There is, however, a way to change it, by adding a `=>` in front of the return type.

```scala
def id(x: Int, y: => Int) = x

id(1, infiniteLoop)
//-> 1
```  

## The difference between `val` and `def`

It seems obvious that `val` defines an immutable value while `def` is used to define a function. But we can write something like

```scala
val sum = a + b
```

The difference from that expression to `def sum(a: Int, b: Int): Int = a + b` is the evaluation method.

`val` uses **CBV**, which means that the evaluation happens in the moment of the definition. While `def` uses **CBN**, and it's evaluation will only happen during the function call.

```scala
def infiniteLoop(): Boolean = infiniteLoop

def x = infiniteLoop
// x: Boolean

val x = infiniteLoop // this will cause an infinite loop
```
