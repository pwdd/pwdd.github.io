---
layout: post
title:  "Working with Options"
date:   2016-10-19 16:21:32 -0500
category: apprenticeship
tags: [scala]
---

In dynamically typed languages, it is a trivial task to write functions that return *something* or *nothing*. In a statically typed language like Scala, *something* does not have the same type as *nothing*. It is possible to check if something exists and handle the exception, but this is not exactly a good way to solve that kind of problem. Having an [`Option`](http://www.scala-lang.org/api/2.11.8/index.html#scala.Option) is. <!--more-->

A Clojure example of a function that returns *something* or `nil`:

```clojure
; clojure
(defn divide
  [a b]
  (if (> b 0)
    (/ a b)))

; 'divide' returns an Integer
(divide 4 2)
;-> 2

; or nil
(divide 1 0)
;-> nil

(defn add
  [a b]
  (+ a b))

; the returned Integer can be passed to other functions
(add (divide 4 2) 2)
;-> 4
```

In Scala, when we explicitly state that `divide` returns *something* or `null`, *something* will not be of type `Int`.

```scala
// scala
def divide(a: Int, b: Int) = if (b > 0) a / b else null

divide(4, 2)
//-> Any = 2

divide(1, 0)
//-> Any = null
```

This would make it difficult to pass the result of `divide` to other functions:

```scala
def add(a: Int, b: Int): Int = a + b

add(divide(4, 2), 2)
// error: type mismatch;
// found : Any
// required: Int
```

## Meet `Option`

The example above is the perfect situation in which `Option` would be up for good use. An *optional*, that sometimes is referred to as "maybe type" because of its implementation in Haskell ([`Maybe` type](https://hackage.haskell.org/package/base-4.9.0.0/docs/Data-Maybe.html)), **encapsulates** the result of an expression that can return *something* or *nothing*.

If `divide` results in an `Int` or nothing, we can say it returns `Option[Int]`.

```scala
def divide(a: Int, b: Int): Option[Int] = if (b > 0) Some(a / b) else None

divide(1, 0)
//-> None

divide(4, 2)
//-> Some(2)
```

Using `Option` would give us the same problem as `Any`. `Some` is not `Int`, so it cannot be argument to functions that require `Int`. However, it is easier to destructure an `Option` and it is a more idiomatic way of dealing with the *something* or *nothing* problem.

### `Option` as a collection

The most obvious way to destruct an `Option` is to use pattern matching.

```scala
divide(4, 2) match {
  case Some(value) => value
  case _ => -1
}
//-> Int = 2
```

Using pattern matching is fine, but `Option` give us a lot more power, because it can be treated as a collection and, thus, we can use high order functions to manipulate its values.

Considering that all our program does is to get the first element of a List of Integers and find the correspondent value in a List of Strings

```scala
// assuming that a List[String] is not empty and
// List[Int], if not empty, has valid indexes
val ls = List("a", "b", "c")

def getElement(li: List[Int]): Option[String] = {
  val index = li.headOption
  index.map(ls(_))
}

val liA = List(2, 0, 1)

getElement(liA)
//-> Option[String] = Some(c)

val liB = List()

getElement(liB)
//-> Option[String] = None
```

Understanding the `getElement`:

* It uses `headOption` instead of `head` to get the first element of a list. `headOption` returns an `Option` and is a safer choice if we are not sure if the list is empty.

```scala
List().head
// NoSuchElementException: head of empty list

List().headOption
// Option[Nothing] = None
```

* `map` being sent to an `Option` shows that we can treat `Option` like a collection. In the `getElement` example, for each element is `index` (an `Option`), get the given element from `ls` (a `List[String]`).

Another examples when `Option` responds to functions from the collection API:

```scala
val lA = List(1, 2, 3).headOption
lA.isEmpty
//-> Boolean = false

val lB = List().headOption
lB.isEmpty
//-> Boolean = true

val listOfOptions = List(Some(1), Some(2), None, Some(3), None)
listOfOptions.flatten
//-> List(1, 2, 3)
```

`Option` also has its own methods. The simplest way to get something or return a default value is to use `getOrElse`

```scala
def getHead(l: List[Int]) = l.headOption.getOrElse(-1)

getHead(List(1, 2, 3))
//-> Int = 1

getHead(List())
//-> Int = -1
```

Note that the default value if nothing is found is an Integer. If we pass it a String, we would have a return type of `Any`:

```scala
def getHead(l: List[Int]) = l.headOption.getOrElse("not found")

getHead(List(1, 2, 3))
//-> Any = 1

getHead(List())
//-> Any = not found
```
