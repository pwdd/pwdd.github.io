---
layout: post
title:  "Functional Programming Design - Data Structures"
date:   2017-03-27 10:50:35 -0500
category: apprenticeship
tags: [scala]
---

When we start thinking about how to represent data in Object Oriented programming, we usually think about a way to make it extensible, so that we can use some design principles that will allow us to make changes to the program without changing existing code. With functional programming, that can be a little bit trickier. <!--more-->

We need to know data structures and how to best use them, instead of creating objects whose behavior we can define. There is no other way around: we need to know what is the difference between call-by-name and lazy evaluation, how to choose between a list, a vector and an array, and why calling `map` on a vector is more efficient than calling `map` on a list. Of course, only having the knowledge of data structures does not make us better programmers &mdash; but it helps.

## Principles and Patterns

There is a joke that has been around for a while comparing design principles and patterns in OO and FP:

|--------------------------------------|---------------------------|
|             OO                       |               FP          |
|:------------------------------------:|:-------------------------:|
| Single Responsibility Principle      |           Functions       |
|--------------------------------------|---------------------------|
| Open/Closed Principle                |           Functions       |
|--------------------------------------|---------------------------|
| Interface Segregation Principle      |           Functions       |
|--------------------------------------|---------------------------|
| Dependency Inversion Principle       |           Functions       |
|--------------------------------------|---------------------------|
| Factory Pattern                      |           Yes, functions  |
|--------------------------------------|---------------------------|
| Strategy Pattern                     |           Functions again |
|--------------------------------------|---------------------------|
| Decorator Pattern                    |           Functions       |
|--------------------------------------|---------------------------|
| Visitor Pattern                      |           Functions       |
|--------------------------------------|---------------------------|


If you still have some patience for polemicists, the above comparison might be fun &mdash; and it should be understood only as a joke.

Imperative and functional programming exist to solve different kind of problems, but aiming for an extensible, modular, easy to change program should be the goal of programmers following either of those paradigms. While the design patterns might differ from OO to FP, SOLID principles do apply to FP. Of course each function, namespace and module should have a single responsibility, and, as long as a language allows some kind of polymorphism (generics or subtyping), we should care about all the other principles.

## The `n` queens problem

As an example of functional design, we can take a look at the [8 queens problem](https://en.wikipedia.org/wiki/Eight_queens_puzzle), in which the queens must be placed on a chess board and none of them can be attacking each other. The solution is usually generalized to solve the problem in which `n` queens are placed on a board of `n x n` dimension.

Before thinking about an algorithm to solve it, it is necessary to think about how to represent the game.

If we had to solve the problem in Ruby, how would we define the board/positions? It could be an `Array` of coordinates (`[[0,0], [0,1], [0,2], ...]`). We could also use two arrays (rows and columns). We could have a `Struct` or an object `Board` with `:row` and `:column` members. What about the queens? Should they be symbols, strings or objects? And how we represent the final result, that must include all the possible combinations for a given number of queens? Would it be a 2D array of coordinates? Should it be a map in which a coordinate is the key and queen/no-queen is a value?

Overthinking the problem is a common misstep, no matter if we are using a functional or an OO language. In the case of OO, it is easy to think that *extensible* means *creating a class so that it can be extended*. Not that functional programming cannot present the same risk. It is easy to try to emulate an object by creating a map and associating data and behavior to it &mdash; which completely defeats the purpose of FP.

So, what if we had to solve the `n` queens problem in a functional language? Which data structure we would use?

This is a slightly adapted version of the solution published in [Programming in Scala](https://booksites.artima.com/programming_in_scala_2ed/examples/html/ch23.html#sec2))

```scala
object Problem {
  type Column = Int

  def solutionFor(numberOfQueens: Int): Set[List[Column]] = {
    def placeQueens(n: Int): Set[List[Column]] =
      if (n == 0) Set(List())
      else
        for {
          // backtracking: for each partial solution
          partialSolution <- placeQueens(n - 1)
          // for each column in the board
          column <- 0 until numberOfQueens
          // is safe to place a queen in a given column?
          if isSafe(column, partialSolution)
          // if true, add the column to the partial solution
        } yield column :: partialSolution
    placeQueens(numberOfQueens)
  }

  def isSafe(col: Int, partialSolution: List[Column]): Boolean = {
    val row = partialSolution.length
    // associate columns (placed queens) with rows
    val queensWithRow = (row - 1 to 0 by -1) zip partialSolution
    // check that placing one more queen keeps everything safe
    queensWithRow forall {
      // the column to be added cannot exist in the partial solution and
      // (math.abs(col - c) != row - r) - the column cannot be attacked
      case (r, c) => col != c && math.abs(col - c) != row - r
    }
  }
}

Problem.solutionFor(4)
//-> Set(List(2, 0, 3, 1), List(1, 3, 0, 2))

//     0   1   2   3                0   1   2   3
//   |---|---|---|---|            |---|---|---|---|
// 0 |   |   | Q |   |          0 |   | Q |   |   |
//   |---|---|---|---|            |---|---|---|---|
// 1 | Q |   |   |   |          1 |   |   |   | Q |
//   |---|---|---|---|            |---|---|---|---|
// 2 |   |   |   | Q |          2 | Q |   |   |   |
//   |---|---|---|---|            |---|---|---|---|
// 3 |   | Q |   |   |          3 |   |   | Q |   |
//   |---|---|---|---|            |---|---|---|---|
```

The beauty of the code is in its simplicity. Three functions solved everything and there was no need to create objects to behave in a certain way. It also uses the [Backtracking algorithm](https://en.wikipedia.org/wiki/Backtracking) &mdash; that is recursive and, thus, the usage of lists as the data structure. The final solution is a `Set[List[Int]]` representing a set of the columns where the queens are placed (the rows are, of course, sequential, and that's why there was no need to use a `List[(Int, Int)]`).
