---
layout: post
title:  "Tail recursion in Scala"
date:   2016-10-05 21:57:32 -0500
category: apprenticeship
tags: [scala]
---

Clojure has a special way to guarantee the tail recursion optimization, with the macro `recur`. In Scala, we can call the function by its name, but there is also a special notation to make sure that the optimization will happen. <!--more-->

## What is tail recursion?

First, it is important to understand what a tail recursion is. It might be easier if we see it:

```scala
// traditional recursion
def sumAll(limit: Int): Int = {
  if (limit == 1) 1
  else limit + sumAll(limit - 1)
}
```

The above implementation of `sumAll`, that sums all numbers from 1 to the `limit`, is a traditional recursion, not a tail recursion, because it performs a sum (`limit + sumAll(limit - 1`), instead of calling the function directly.

A tail recursive definition would be like this:

```scala
// tail recursive
def sumAll(number: Int): Int = {
  def sum(base: Int, acc: Int): Int = {
    if (base == 1) acc
    else sum(base - 1, acc + base)  
  }
  sum(number, 1)
}
```

This time, the private function `sum` call itself and is tail recursive.

Tail recursion is a subroutine that calls a function as the final action of a procedure. When the subroutine makes the call more than once, those calls can be optimized to use only one [stack](https://8thlight.com/blog/doug-bradbury/2015/04/27/stack-overflow.html), instead of creating new ones and taking the risk of causing a Stack Overflow.

It does not mean that everytime we write a function/method that calls itself in the end, it will be optimized. Functional languages, like Clojure and Scala, guarantee the optimization, though.

In Clojure, the way to guarantee it is to use the macro `recur`, instead of the name of the function:

```clojure
; tail recursive
(defn factorial
  [number]
  (loop [base number
         acc 1]
    (if (= base 1)
      acc
      (recur (dec base) (* base acc)))))

(factorial 5)
;-> 120
```

In Scala, it would be:

```scala
//tail recursive
def factorial(number: Int): Int = {
  def fac(base: Int, acc: Int): Int = {
    if (base == 1) acc
    else fac(base - 1, acc * base)
  }
  fac(number, 1)
}

factorial(5)
//-> 120
```

## Check for tail recursion

The optimization will happen automatically, but if we need to make sure that a method is tail-recurse, we can put `@tailrec` before the method definition:

```scala
import scala.annotation.tailrec

def factorial(number: Int): Int = {
  @tailrec def fac(base: Int, acc: Int): Int = {
    if (base == 1) acc
    else fac(base - 1, acc * base)
  }
  fac(number, 1)
}

factorial(5)
//-> 120
```

If the method was not tail-recursive, it would throw an error at compile time:

```scala
import scala.annotation.tailrec

@tailrec
def factorial(number:Int) : Int = {
  if (number == 1) 1
  else number * factorial (number - 1)
}

factorial(5)
// <console>: error: could not optimize @tailrec annotated method factorial: it contains a recursive call not in tail position
//            else number * factorial (number - 1)
//                        ^
```

The error shows that the `factorial` function does not have a tail recursive call at the end. If we had not used the `@tailrec` annotation, we would get a `StackOverflow` error when trying to use the function on big numbers.
