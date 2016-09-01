---
layout: post
title:  "A 5x5 board"
date:   2016-09-01 01:45:41 -0500
category: apprenticeship
tags: [clojure]
---

One of the most interesting things that can happen while working on a project is dealing with the unknown. So, everybody knows how to play a Tic Tac Toe and knows how it works. But, what if we play with symbols other than *x* and *o*? What if we can give the option to save the game? What if we play with a 5x5 board?<!--more-->

After having the game working on a 3x3 board, I was asked to allow the user to choose playing in a 4x4 and a 5x5 board. The visual representation of the board had to be changed to include numbers where, before, I had only white spaces. And the biggest change: improving the minimax performance.

## The first try: limiting depth

The first thing to do was to have a limit depth, so that the search would stop after a while.

```clojure
(if (or (game-over? board)
        (>= depth limit-depth))
  (board-analysis board))
```

It worked, but it was still very slow for limit depths higher than 4.

I must say that I tried to implement the [Alpha Beta Pruning](https://en.wikipedia.org/wiki/Alpha%E2%80%93beta_pruning), but I could not find a nice way to do it with Clojure. I would probably have to re-write the minimax function so that I would be able to pass alpha and beta around. It is still in my personal list of todos.

## The second try: randomize initial moves

With such a small limit depth, the first moves were not that smart &mdash; they were actually sequential. It would avoid the opponent from winning and winning when possible, but it was not smart other than that. So, I decide to randomize the initial moves and then handle it back to the minimax.

In a 4x4 board, it would wait for 5 moves (counting both the computer and the opponent). This means that, if opponent started, it would be able to fill in 3 positions in a row/column/diagonal. The minimax would come in to block it from winning. In a 5x5, it would take 7 moves.

```clojure
(cond
  (board/is-board-empty? board) (rules/place-in-the-center board)
  (starting-game-with-alternative-board? board) (helpers/random-move board)
  :else
    (run minimax))
```

## The final: mix medium and hard computer

The solution above was acceptable: it was still an unbeatable computer. However, it does not mean that it was a very smart one. What I did to improve it was to use the function that gets the [medium computer move]({% post_url 2016-09-01-medium-difficulty-level-on-TTT %}).

That means that the first moves would be based on a set of rules. When it reached a certain point in the game, it would handle back to the minimax. This way, I could also make the limit depth higher.

```clojure
(cond
  (board/is-board-empty? board) (rules/place-in-the-center board)
    (starting-game-with-alternative-board? board)
      (rules/play-based-on-rules player params)
    :else
      (run minimax))
```

Now, the computer is still unbeatable and a little bit smarter. The game runs without delay, which was the original goal.

I've never played in a 4x4 or 5x5 board before, so I'm not sure about forks. That's why I'll probably keep looking into implementing the pruning and trying other solutions.
