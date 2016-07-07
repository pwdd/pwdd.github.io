---
layout: post
title:  "The Case of the Mysterious AI"
date:   2016-07-07 11:34:46 -0500
category: apprenticeship
tags: [clojure]
---

The minimax algorithm assumes that both players make the best possible move they can. What if one of the players makes only incredibly bad moves? <!--more-->

During the implementation of the [minimax](http://pwdd.github.io/post/minimax-in-clojure), I could not understand why the unbeatable computer didn't behave as expected when playing against bad players. When it played against the easy computer (that would make random but valid moves), it would lose in some situations. That was obviously a bug that needed fixing &mdash; and it was reasonably easy to find out what was causing the problem.

It was not in the minimax implementation, but in a function used to create `Player` records. `make-player` would create a `Player` with value `1` if it was an AI, and `-1` if it was not. Because of that, both the easy computer and the unbeatable computer had value `1`. That was breaking the [base case](http://pwdd.github.io/post/minimax-in-clojure/#the-base-case), that should return the board evaluation from the current player perspective.

After fixing that, it was time to work with a more mysterious bug: the unbeatable computer would not loose, but instead of making the winner move as soon as possible, it would take the game to the last state. For instance, in the following board (unbeatable computer is 'o'):

```
 o | x | x
---|---|---
 4 | o | 6
---|---|---
 7 | x | 9  
```

The unbeatable computer would not pick position 9. It would pick the 4th one, so the game would go on. It would win, but it was not *that* smart. I tried changing the `negamax` function, then the `best-move`, and nothing would work. My guess was that there was something wrong with the `depth` used in the board analysis.

My guess was right, but not for the right reason. The longer the unbeatable computer took to win, the highest was the value of a position, so I thought that making the `depth` decrease instead of increase would fix the problem. It did, but it broke something else. Then, I thought about changing the base case to consider if the game was over and if `depth` was no greater than `1`. It broke everything.

I put the problem aside and started working on other features that needed to be implemented. By the end of the day, I started reviewing the tests, to make sure they had clear messages. When I was reading the tests for the `board-analysis`, I found out what was the problem.

```clojure
; wrong implementation
(defn board-analysis
  [board unbeatable-computer opponent depth]
  (let [winner (winner board)]
    (cond
      (= winner unbeatable-computer) (+ 10 depth)
      (= winner opponent) (- 10 depth)
      :else
        0)))
```

That means that, if the unbeatable computer won a game, it would return, let's say, `13`, while for the opponent, the same game would have the value `7`. That completely changed the logic of the game, that was not zero sum anymore! The right implementation:

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

In the same scenario, if the computer won the game, it would return `7`, while the opponent would get the value `-7`.

## Decrypting error messages

One of the most upsetting things for me when I started learning Clojure was the cryptic error messages. If something broke, dozens of lines of would be outputted to the console, none that I could understand. While implementing the minimax, some of them were so frequent that I had no option other them learn what they meant.

- `IllegalArgumentException Key must be integer`

In the `best-move` function, the returned value comes from evaluating the form `(nth available-spots index-of-the-highest-score)`. And `index-of-the-highest-score` comes from getting all the values returned by the minimax function. If anything went wrong with the minimax, `index-of-the-highest-score` would not be an integer, and ["Key must be integer"](https://github.com/clojure/clojure/blob/d5708425995e8c83157ad49007ec2f8f43d8eac8/src/jvm/clojure/lang/APersistentVector.java#L289) will happen.

- `clojure.lang.LazySeq cannot be cast to java.lang.Number`

At first, I tried to use `for` to get the minimax number, which ended up not working. I expected the function to return an integer, but I was getting back a lazy sequence instead. When I tried to compare the value with some other integer, it would go wrong. If, for instance, the `minimax-score` was `(lazy-seq 1)` instead of `1`, at some point I would be evaluating `(max 2 (lazy-seq 1))`, that clearly does not work.

- `No such namespace`

A namespace living in a file that was not correctly compiled will cause the problem. For instance, if the namespace `board` has a compiling error and is required in `game` namespace, the error message will say that there is ["No such namespace"](https://github.com/clojure/clojure/blob/e8c72929f9648f99e7914e939602c7bdd7928022/src/jvm/clojure/lang/Compiler.java#L7153).
