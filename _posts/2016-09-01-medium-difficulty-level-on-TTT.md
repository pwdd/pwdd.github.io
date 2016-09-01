---
layout: post
title:  "Adding a medium difficulty to TT"
date:   2016-09-01 11:47:41 -0500
category: apprenticeship
tags: [clojure]
---

After adding an easy computer and an unbeatable computer to the Tic Tac Toe game, it was difficult to think was a *medium* computer would be like, how it would behave, what kind of moves it would make.<!--more-->

It was defined on my IPM with my mentors that it would 1) win when it had the chance to; 2) block opponent from winning; and 3) not be able to avoid *forks*. The idea was to write it using a rule-based algorithm.

## The rules

I created a namespace to hold the functions related to the rules that the medium computer should follow. The first thing to do was to create a function that would take the indexes of a row, a column or a diagonal and get the correspondent combo from the board.

```clojure
(defn- correspondent-board-combo
  [board indexes-combo]
  (mapv #(nth board %) indexes-combo))
```

From there, it was just a matter of writing which one of the rules. As I had used a multimethod to get the move of a player based on its role, there was not a lot of chances to be done. This is how the medium player `select-spot` looks like:

```clojure
(defn play-based-on-rules
  [player board current-player-marker opponent-marker board-size]
  (cond
    (is-center-the-best-move? board) (place-in-the-center board)
    (is-corner-the-best-move? board) (place-in-a-corner board)
    (can-win? board current-player-marker)
      (place-in-winning-spot board current-player-marker)
    (can-win? board opponent-marker)
      (place-in-winning-spot board opponent-marker)
    (owns-combos? board current-player-marker opponent-marker)
      (fill-in-a-combo board current-player-marker opponent-marker)
    (is-there-empty-combos? board)
      (helpers/random-move (get-an-empty-combo board))
    :else
      (helpers/random-move (board/available-spots board))))
```
