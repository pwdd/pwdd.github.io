---
layout: post
title:  "Multimethods in multiple namespaces"
date:   2016-08-09 9:54:35 -0500
category: apprenticeship
tags: [clojure]
---

One of the multimethods that I use on the Tic Tac Toe project is related to how players select a spot on the board. The hardest thing to do while implementing the methods was, actually, dealing with dependencies.<!--more--> I had to create a separated namespace, `ttt.get-spots` that would hold only `select-spot` and its methods.

`ttt.get-spots` ended up depending on many other namespaces, mostly those related to each player. It needed `ttt.negamax` to handle the unbeatable computer, `ttt.prompt` to deal with the user input, among others. The `ttt.get-spots` was then required in the namespace that handle the game loop.

## Concrete vs abstract

My first thought was that, if I had the concretion (`defmethod`) separated from the abstraction (`defmulti`), I would end up using conditionals to call the methods: if player is unbeatable computer, use `ai/select-spot`, if player is user...

I was wrong. It is possible to separate them and call only `select-spot`, without specifying the namespace that each method comes from, but only the one where `defmulti` is. Anything that uses `select-spot` needs to know only about the abstraction.

Here is how the separation worked out:

**get_spots.clj**

```clojure
(ns ttt.get-spots)

(defmulti select-spot (fn [player & params] (:role player)))
```

**prompt.clj**

```clojure
(ns ttt.prompt
  (:require [ttt.get-spots :refer [select-spot]]))

(defmethod select-spot :human
  [player params])
```

**negamax.clj**

```clojure
(ns ttt.negamax
  (:require [ttt.get-spots :refer [select-spot]]))

(defmethod select-spot :hard-computer
  [player params])
```

**easy_computer.clj**

```clojure
(ns ttt.prompt
  (:require [ttt.get-spots :refer [select-spot]]))

(defmethod select-spot :human
  [player params])
```

And the namespace that uses `select-spot`:

```clojure
(ns ttt.game-loop
  (:require [ttt.get-spots :as spots]))

(defn game-loop
  [player params]
  ; ...
  (spots/select-spot player paramas))
```

## When to separate them

I use multimethods in other situations, but I didn't separate them. One case is, for instance, the final message of the game, that also depends on what role the winner. The idea is that, when we want to protect those who use that namespace from the details of the implementation, we separate them. Otherwise, when the multimethod is more like an alternative syntax for `cond`, they can stay at the same place.
