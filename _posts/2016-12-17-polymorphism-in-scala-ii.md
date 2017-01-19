---
layout: post
title:  "Polymorphism in Scala - II - and the Liskov Substitution Principle"
date:   2016-12-17 14:50:35 -0500
category: apprenticeship
tags: [scala]
---

When dealing with generic polymorphism and type parameterization, we can set some limitation to which type a method can accept. <!--more-->

## Bounds

Assuming we have the types `Empty` and `NonEmpty`, and both extend `IntSet`.

```
       ----------
      /| IntSet |\
     / ---------- \
____/____      ____\_______  
| Empty |      | NonEmpty |
---------      ------------
```

If we needed to filter a set of integers to return a new set of integers, we could say that: **1)** if the set is `Empty`, it returns an `Empty` set; **2)** if the set is `NonEmpty`, returns a `NonEmpty` set or throws an exception. This means that a function to get all positive numbers from an `IntSet` can accept a subtype of `IntSet` as parameter (it can accept an `Empty` or a `NonEmpty`).

```scala
def getAllPos(S <: IntSet)(arg: S): S = // implementation
```

`IntSet` is the **upper bound** for the types that `allPos` accepts. That is what `<:` denotes: `S` is a subtype of `IntSet`.

It is also possible to define a lower bound or both. For instance, we could say that a method accepts `S >: NonEmpty <: IntSet` &mdash; which means that it accepts a `S` that is a subtype of `IntSet` and a super-type of `NonEmpty`.

## Covariance

What if we need to use those types in a collection? Could we say that `List[S] <: List[IntSet]` &mdash; a list of `S` is also a subtype of a list of `IntSet`? We could. This would be a [**covariant**](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)) relationship.

### The Liskov Substitution Principle

The problem with covariance is that, depending on how it is implemented, it can lead to illegal situations. And there is where the [Liskov Substitution Principle &mdash; LSP](https://en.wikipedia.org/wiki/Liskov_substitution_principle) is important.  

The principle defines when a type can be a subtype of another. It says that if `A` is a subtype of `B`, then `A` should be able to do everything `B` does.

In Java, arrays are covariant, and in certain occasions we can violate the LSP.

```java
// In Java, arrays are covariant
NoEmpty[] a = new NonEmpty[] { new NonEmpty(1, Empty, Empty) };

// set by reference ('a' and 'b' points to the same place in memory)
IntSet[] b = a;

// b[0] is Empty and, because it was set by reference, a[0] is also Empty
b[0] = Empty;

// 's' is 'Empty'
NonEmpty s = a[0];
```

The Java example above would throw the `ArrayStoreException` at runtime, because in the last line we are setting an `Empty` value (`a[0]`) to a `NonEmpty` (`s`).

```scala
// In Scala, arrays are NOT covariant
val a: Array[NonEmpty] = Array(new NonEmpty( 1, Empty, Empty))
val b: Array[IntSet] = a
b(0) = Empty
val s: NonEmpty = a(0)
```

The Scala example would throw an exception at the compiling time, when we tried to define `b`, because `Array[NonEmpty]` is not a subtype of `Array[IntSet]`. Arrays are not covariant in Scala. The reason why lists can be covariant while arrays cannot is because lists are immutable.

### Defining covariants

When creating our own classes, we can determine if a class is covariant.

Assuming we have two types, `A` and `B`. We can have the following relationship between then:

```scala
// 1
A <: B // 'A' is a subtype of 'B' => covariance

// 2
A >: B // 'A' is a supertype of 'B' => contravariance

// 3
// there is no relationship between 'A' and 'B' => nonvariance
```

If we have to create classes that accept `A` or `B` as parameter, we can make it clear when the classes will also have a relationship

```scala
// 'Foo' is covariant
class Foo[+A] { }

// 'Foo' is contravariant
class Foo[-A] { }

// 'Foo' is nonvariant
class Foo[A] { }
```

The rules of when we can create a covariant or contravariant class are such that *covariants can be used if the type parameter only appears as the result of the methods* and *contravariants can be used if the type parameter is used as arguments of the methods*. The compiler checks for the rules so we do not need to remember them.

We can write the immutable linked-list example used in the [first post about polymorphism](% post_url 2016-12-16-polymorphism-in-scala-i %) to use covariance, so that `Nil` does not need to be a class. After all, we do not need new instances of `Nil`, so it should be object &mdash; and, as an object, it cannot be parameterized.

```scala
trait List[+T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  def isEmpty: Boolean = false
}

object Nil extends List[Nothing] {
  def isEmpty: Boolean = true
  def head: Nothing = throw new NoSuchElementException
  def tail: Nothing = throw new NoSuchElementException
}
```

`Nothing` is a subtype of any other Scala type, and as `T` is a covariant, the relationship is possible.

```scala
// List of Nothing is a subtype of List of String
val nil: List[String] = Nil
```

And, if we wanted to add a method to prepend an element to a List without violating the LSP, we could rewrite trait `List[+T]` like this:

```scala
trait List[+T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
  def prepend[U >: T](element: U): List[U] = new Cons(element, this)
}
```

This is possible because `U` is a supertype of `T`. If we had written

```scala
def prepend(element: T): List[T] = new Cons(element, this)
```

it would break the type checking at compiling time, because as `List[+T]` is covariant, the type should not be used as the type of the parameter of a method.

Back to our `IntSet` type and its subtypes:

```
       ----------
      /| IntSet |\
     / ---------- \
____/____      ____\_______  
| Empty |      | NonEmpty |
---------      ------------
```

Let's say that we have a `x`, that is a `List[IntSet]`, and `y`, a `List[NonEmpty]`. While we could write

```scala
// x: List[IntSet]
// prepend Empty to an List[IntSet] is OK
// Empty is a IntSet
x.prepend(Empty)
```

the same would not be true to

```scala
// y: List[NonEmpty]
// prepend an Empty to a List[NonEmpty] throws Exception
// Empty is not NonEmpty
y.prepend(Empty)
```

Which means that it violates the LSP: although `Empty` and `NonEmpty` are subtypes of `IntSet`, they cannot do everything `IntSet` does.

By stating that the element to be prepended to a `List[+T]` should be a supertype of `T`, we make it possible that the two situation above compile properly.

Using the definition `def prepend[U >: T](element: U): List[U] = new Cons(element, this)`, prepending `Empty` to a `NonEmpty` list works:

```scala
// y: List[NonEmpty]
// def prepend[U >: T](element: U): List[U] = new Cons(element, this)
y.prepend(Empty)
```

What has changed:

- `y` is a `List[NonEmpty]`, which means that `T` is `NonEmpty`
- `U` would be `Empty`, that is an `IntSet` &mdash; and an `IntSet` is a supertype of `NonEmpty`, which makes it possible to pass it as an argument to `prepend`.
