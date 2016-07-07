---
layout: post
title:  "Minimax in Clojure"
date:   2016-07-02 16:44:27 -0500
category: apprenticeship
tags: [clojure]
---

It finally came the day when I was asked to implement an unbeatable player in the Tic Tac Toe project. I had (horribly) done it before in Ruby, so I thought it would be reasonably easy: I knew the algorithm already and I thought I had a good grasp of how recursion works in Clojure. What could go wrong? Everything. <!--more--> It took me **a lot** more time than I expected to figure things out.

## The minimax algorithm

There are many posts and documents out there that explain the [minimax](https://en.wikipedia.org/wiki/Minimax) algorithm very well. Away from the theory, one of my favorites is this [video](https://www.youtube.com/watch?v=J1GoI5WHBto), that shows the tree evaluation step by step.

Simply put, in a [zero sum](https://en.wikipedia.org/wiki/Zero-sum_game) game, both players make the best moves they can. The *max* player always makes the move that will *maximize* its chance to win, while the *min* player always makes the move that will *minimize* the chance of *max* to win.

In the [negamax](https://en.wikipedia.org/wiki/Negamax) algorithm &mdash; that is just a variation of the minimax &mdash; both players are *max* and the **value of a move is given according to the current player point of view**. This was the algorithm that I tried to implement.

## The base case

In order to get a position value, from the current player perspective, it is necessary to give each player a value. The unbeatable computer has the value `1`, while the opponent has the value `-1`. The base case is the game over, and the returned value is the analysis of the board multiplied by the value of a player.

Considering the following situation (computer is 'o'). It is now time for the computer to play:

```
 o | x | 3
---|---|---
 4 | x | 6
---|---|---
 7 | 8 | 9  
```

If the computer chooses any position other than 8, it gives the chance for the opponent to win. For instance, let's say the computer picks the position 4. It does not win the game, so the value would be, let's say, `0`. Then, it is time for the opponent to play &mdash; and it chooses the position 8. The game is over. That means that, for the computer, the value of the game when it is over could be `-10` (it loses) and, for the opponent, `10`. That is why we give the opponent the value `-1`, so when the computer is evaluating the board, `(* -1 -10)` becomes `10`, and the computer knows that 8th is a valuable position (it does not win the game, but avoids the opponent's victory).

The base case can be written like this:

```clojure
(if (game-over? board)
  (* (value current-player) (board-analysis board))
  (analyze-next-position))
```

## The board analysis

The analysis of the board can be done in many ways. The important thing is to maintain the idea that, if a board has the value `+Infinity` to one player, the same board necessarily has the value `-Infinity` to the other player.

One of the things that can be added to make the computer *smarter* is the value of the `depth`. Considering this board (computer is 'o'):

```
 o | x | 3
---|---|---
 4 | o | 6
---|---|---
 7 | x | 9  
```

The computer can move to position 9 and win the game, but it can also keep making moves until the board is full. It will *probably* win, but it takes the risk to tie the game.

With `depth`, the board will have different values depending on how deep the evaluation goes. The earlier the victory, the more valuable the position.

The board analysis could be written like this:

```clojure
(defn board-analysis
  [board unbeatable-computer opponent depth]
  (let [winner (winner board)]
    (cond
      (= winner unbeatable-computer) (- 10 depth)
      (= winner opponent) (- depth 10)
      :else
        0)))
```

# How to get the best position

The negamax function will return the value of a move that will give the unbeatable player the highest score:

```clojure
(defn negamax
  [board current-player opponent depth]
  (if (game-over? board)
    (* (value current-player) (board-analysis board))
    (let [spots (available-spots board)
         new-boards (map #(move board current-player %) spots)]
      (apply max (map #(- (negamax %
                                   opponent
                                   current-player
                                   (inc depth)) new-boards))))))
```

The value is there, but now what? How is it possible to know which position that value is associated to?

In order to find it, I had to find all the scores, not only the highest one:

```clojure
(defn scores
  [board current-player opponent depth]
  (let [spots (available-spots board)
       new-boards (map #(move board current-player %) spots)]
    (map #(- (negamax % opponent current-player (inc depth)) new-boards))))
```

From there, it is possible to get the best move:

```clojure
(defn best-move
  [board current-player opponent depth]
  (let [
         spots (available-spots board)
         scores (scores board current-player opponent depth)
         highest-score (apply max scores)
         index-of-highest-scores (.indexOf scores highest-score)
       ]
    (nth spots index-of-highest-scores)))
```

## Two last things

One thing to notice from the functions above is that `negamax` and `scores` have repeated code. This gets even clearer inside the `best-move` function, when it is only necessary to apply `max` to `scores`, and not to call `negamax` in order to get the highest value. From there, it is possible to do:

```clojure
(declare negamax)

(defn scores
  [board current-player opponent depth]
  (let [spots (available-spots board)
       new-boards (map #(move board current-player %) spots)]
    (map #(- (negamax % opponent current-player (inc depth)) new-boards))))

(defn negamax
  [board current-player opponent depth]
  (if (game-over? board)
    (* (value current-player) (board-analysis board))
    (apply max (scores board current-player opponent depth))))
```

The other thing that I learned while doing this was about [`memoize`](http://clojure.github.io/clojure/clojure.core-api.html#clojure.core/memoize). If a function is [referentially transparent](https://en.wikipedia.org/wiki/Referential_transparency), it can be cached, so consequents calls to the same function with the same arguments return the cached result. `(def negamax (memoize negamax))` improved the performance considerably.

```clojure
; non-memoized
(time (best-move [:o :_ :_
                  :_ :x :_
                  :_ :_ :x]
                  unbeatable-computer
                  opponent
                  0)))))
; "Elapsed time: 63.800254 msecs"
; 2

; memoized
(time (best-move [:o :_ :_
                  :_ :x :_
                  :_ :_ :x]
                  unbeatable-computer
                  opponent
                  0)))))
; "Elapsed time: 0.578481 msecs"
; 2
```

I didn't came up with the `negamax` implementation without looking lots of other implementations. The way recursion works really got me for a long time, specially because I thought I would be dealing with an integer when I was in fact getting a collection. So, after I had it working, I deleted it and wrote it again. Twice. I wanted to make sure that I understood what I was doing and that I was able to write it again. I think that now I do. And I've been secretly thinking how much fun it would be to do the same project using other languages as well, just to see the difference between then.
