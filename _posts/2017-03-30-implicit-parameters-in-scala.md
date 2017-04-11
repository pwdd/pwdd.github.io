---
layout: post
title:  "Implicit Parameters in Scala"
date:   2017-03-30 17:50:35 -0500
category: apprenticeship
tags: [scala]
---

Because Scala compiler can infer types that are not explicitly declared, it also makes it possible to use **implicit** parameters.<!--more-->

Here is a definition of the [merge sort algorithm](https://en.wikipedia.org/wiki/Merge_sort), in which we split in smaller parts and sort them until we are able to merge then back again. <sup>[1](#section)</sup>

The algorithm will be defined three times to show how we can go from explicitly defining a parameter and how we can finally let it implicit.

```scala
// 'lt' (less than) compares two elements
def mergeSort[T](xs: List[T])(lt: (T, T) => Boolean): List[T] = {
  val n = xs.length/2
  if (n == 0) xs
  else {
    def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
      case (Nil, _) => ys
      case (_, Nil) => xs
      case (x :: xs1, y :: ys1) =>
        if (lt(x, y)) x :: merge(xs1, ys)
        else y :: merge(xs, ys1)
    }
    val (fst, snd) = xs splitAt n
    merge(mergeSort(fst)(ord), mergeSort(snd)(ord))
  }
}

val nums: List[Int] = List(2, -4, 5, 7, 1)
val fruits: List[String] = List("apple", "pineapple", "orange", "banana")

mergeSort(nums)((x: Int, y: Int) => x < y)
//-> List(-4, 1, 2, 5, 7)
mergeSort(fruits)((x: String, y: String) => x.compareTo(y) < 0)
//-> List(apple, banana, orange, pineapple)
```

## A more general sorting

The way it is defined above, `mergeSort` would require us to always be passing in a specific function every time we call it. `x < y` for sorting a list of integers, `compareTo` if we are dealing with strings and so on.

We can get skip the function definition all together by using [Ordering](http://www.scala-lang.org/api/2.12.0/scala/math/Ordering.html), a library with methods to do the sorting.

```scala
import math.Ordering

// 'ord' is an element of type Ordering[T]
def mergeSort[T](xs: List[T])(ord: Ordering[T]): List[T] = {
  val n = xs.length/2
  if (n == 0) xs
  else {
    def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
      case (Nil, _) => ys
      case (_, Nil) => xs
      case (x :: xs1, y :: ys1) =>
        // 'lt' is now a method implemented in Ordering library
        if (ord.lt(x, y)) x :: merge(xs1, ys)
        else y :: merge(xs, ys1)
    }
    val (fst, snd) = xs splitAt n
    merge(mergeSort(fst)(ord), mergeSort(snd)(ord))
  }
}

val nums: List[Int] = List(2, -4, 5, 7, 1)
val fruits: List[String] = List("apple", "pineapple", "orange", "banana")

mergeSort(nums)(Ordering.Int)
//-> List(-4, 1, 2, 5, 7)
mergeSort(fruits)(Ordering.String)
//-> List(apple, banana, orange, pineapple)
```

## Cleaning up

The example above is a bit cleaner, but there is still space for more abstractions.

When we call the function, we declare types of the values that are going to be passed to the function, but we could perfectly skip the declaration, since the compiler can infer it

```scala
val nums = List(2, -4, 5, 7, 1)
val fruits = List("apple", "pineapple", "orange", "banana")
```

We also know that `mergeSort` takes a `List[T]` and `ord` is a `Ordering[T]`, which means that, if the compiler can infer the type of `T` when we declare the `val`s `nums` and `fruits`, it should also be able to understand that `T` in `Ordering[T]` is of the same type.

The way we can make it happen is by declaring `ord` as an **implicit** parameter:

```scala
import math.Ordering

// 'ord' is implicit
def mergeSort[T](xs: List[T])(implicit ord: Ordering[T]): List[T] = {
  val n = xs.length/2
  if (n == 0) xs
  else {
    def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
      case (Nil, _) => ys
      case (_, Nil) => xs
      case (x :: xs1, y :: ys1) =>
        if (ord.lt(x, y)) x :: merge(xs1, ys)
        else y :: merge(xs, ys1)
    }
    val (fst, snd) = xs splitAt n
    merge(mergeSort(fst)(ord), mergeSort(snd)(ord))
  }
}

val nums = List(2, -4, 5, 7, 1)
val fruits = List("apple", "pineapple", "orange", "banana")

// there is no need to pass 'ord', that is left implicit
mergeSort(nums)
//-> List(-4, 1, 2, 5, 7)
mergeSort(fruits)
//-> List(apple, banana, orange, pineapple)
```

When a function takes an implicit parameter of type `T`, the compiler will search an implicit definition that:

- is marked `implicit`
- has a type compatible with `T`
- is visible at the point of the function call, or is defined in a companion object associated with `T`

###### 1

[Fundaments of Functional Programming](https://www.coursera.org/learn/progfun1)
