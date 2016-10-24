---
layout: post
title:  "Negamax in Scala"
date:   2016-10-21 14:21:32 -0500
category: apprenticeship
tags: [scala]
---

The [Scala evaluation method]({% post_url 2016-10-14-how-scala-is-evaluated %}) makes it more efficient to deal with immutable values, but it still give us the chance to easily deal with mutable data. It is possible to write a program following the Function Programming paradigm while still being able to rely on mutable object fields. Thanks to that, it is reasonably easy to implement the negamax with alpha beta pruning algorithm, improving the performance of an unbeatable computer in a Tic Tac Toe game. <!--more-->

## Negamax with alpha beta pruning

First things first, what is [negamax](https://en.wikipedia.org/wiki/Negamax)?

It is just a variation of the minimax algorithm.

In the minimax, a player, max, always choose the move that will give it the maximum possibility to win, while another player, min, always choose the move that will minimize the chances of the opponent to win. If a move has the value 100 for max, it will have the value -100 for min (max would choose that in order to win, and min would choose that in order to block max from winning).

In the negamax algorithm, both players are *max* and the search returns the value according to the current player's perspective. The best value for the current player **negates** the best value of the opponent: `max(a,b) = -min(-a, -b)`.

And what is [alpha beta pruning](https://en.wikipedia.org/wiki/Alpha%E2%80%93beta_pruning)?

It is a way to make a tree search shorter. Alpha keeps track of the best value for the current player and beta, for the opponent. If a move is worse than the previous one, it *prunes* the tree, and stop the search. Considering the following board:

The current player is **x**

```shell
 x | x | 3
---|---|---   
 o | o | 6
---|---|---
 7 | 8 | 9
```

The search would find that position 3 has the value 100 (the value returned by the a hypothetical [heuristic analysis of the board](https://en.wikipedia.org/wiki/Evaluation_function)). It means that if current player moves to that spot, its best value would be 100, and alpha would be 100. Then, it searches the next node. It finds that if current player moves to spot 6, its best value would be 50. Alpha &mdash; that is currently holding the value 100 &mdash; is greater than 50, so it would stop the search. This means that the search evaluated 2 nodes, instead of 5.  

## Updating the value of alpha

In Clojure, because it is a purely functional language, trying to emulate the behavior of imperative programming and break out of loop if a condition is met and dealing with a mutable value (updating alpha) is complicated. At least, it was for me, when I tried. I [ended up](https://github.com/pwdd/ttt-clojure/blob/master/src/ttt/computer/negamax.clj#L27) using `map` to generate the boards and run the analysis.

In Scala, because it can be used as a Object Oriented language, dealing with a mutable variable was easier. My solution was to have a `var` associated with the object and update it during the loop.

Two things to consider while writing the algorithm:

1. Values passed as parameters cannot be updated, so it is necessary to create a copy of alpha in order to keep it updated.

2. There are different ways to break out of a loop.

## The solution

Here is a edited version of the solution. Both players start with the worst possible value: alpha is -Infinity and beta is Infinity. The initial value of the best move was -1 (out of the range of the board).

```scala
import scala.util.control.Breaks._

object Negamax {
  var bestMove = -1

  def score(board: List[Symbol],
            currentPlayerMarker: Symbol,
            opponentMarker: Symbol,
            depth: Int = 0,
            alpha: Double = Double.NegativeInfinity,
            beta: Double = Double.PositiveInfinity): Double = {

    var maxScore = Double.NegativeInfinity

    if (isFinalState()) {
      boardAnalysis()
    }

    else {
      var alphaCopy = alpha
      val availableSpots = Board.availableSpots(board)

      breakable {
        availableSpots.foreach { spot =>
          val newBoard = Board.move(board, currentPlayerMarker, spot)
          val negamaxScore = -score(newBoard,
                                    opponentMarker,
                                    currentPlayerMarker,
                                    depth + 1,
                                    -beta,
                                    -alphaCopy)

          if (negamaxScore > maxScore) {
            maxScore = negamaxScore
            if (depth == 0) bestMove = spot
          }

          alphaCopy = Math.max(maxScore, alphaCopy)

          if (alphaCopy >= beta) {
            break()
          }
        }
      }
      alphaCopy
    }
  }
}
```

## Breakable

It is necessary to import `scala.util.control.Breaks` in order to use `break()`. It is also necessary to surround the loop in a `breakable`, so that when the loop breaks, it continues the evaluation outside the loop. Otherwise, it would break the evaluation completely.

## The time

There are many ways to benchmark Scala programs and whole libraries that do it. This is a very simple way, just good enough for this purpose:

```scala
def time[T](block: => T): T = {
  val start = System.currentTimeMillis
  val res = block
  val totalTime = System.currentTimeMillis - start
  println("Elapsed time: %1d ms".format(totalTime))
  res
}

val board = List(
  x, x, x, _,
  o, o, o, _,
  x, _, _, _,
  _, _, _, _)

val computer = new Computer

// current player is 'o'

// negamax with pruning
time { computer.getSpot(board) }
// Elapsed time: 9 ms
//-> Int = 7

// negamax without pruning
time { computer.getSpot(board) }
// Elapsed time: 892 ms
// Int = 7
```
