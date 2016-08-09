---
layout: post
title:  "Implementing Polymorphic Functions"
date:   2016-07-01 11:48:58 -0700
category: apprenticeship
tags: [clojure]
---

How easy would it be to use a `multimethod` that could handle multiple arguments? And what is the point of having such methods? Wouldn't they add complexity to the problem? I deferred answering those questions as much as I could while developing the Tic Tac Toe. <!--more-->

The first step of the project was to build a simple game: 'X' would play first, 'O' would play second, and both players would be *humans*. At each player's turn, `select-spot` would ask for the user input. That was it.

The second step was to add a computer player. At that point, I started thinking about implementing polymorphism, so that I could have just one function able to handle different players: if player had the role `human`, `select-spot` would ask for the user input; if the player had the role `computer`, it would generate a random number.

After learning about protocols, records and multimethods, I realized that it was too much trouble for something that could have a simpler solution. My decision was to identify the player as a `map` that would hold the values of its marker and its role. For instance, `{ :role :human :marker :x }`.

I solved the problem with the following function:

```clojure
(defn select-spot
  [player board-length]
  (if (= :human (:role player))
    (human/get-human-spot)
    (computer/get-computer-spot board-length)))
```

That worked alright until the next step of the project: add an unbeatable computer player. Then, I had not only one, but several functions that should behave differently according to the role of a player.

## Protocols

The first solution I tried was to use a `protocol`, a set of methods that must be implemented by all data structures that extend it. This is a simple example of a `protocol` and how it can be used:

```clojure
; all data structures that use the protocol 'Stringification'
; must implement 'stringify' and 'shout'

(defprotocol Stringification
  (stringify [this])
  (shout [this]))

; extend vectors
(extend-type clojure.lang.PersistentVector
  Stringification
  (stringify [this] (clojure.string/join (map str this)))
  (shout [this] (str (clojure.string/upper-case (stringify this)) "!!!"))))

; extend integers
(extend-type java.lang.Long
  Stringification
  (stringify [this] (str this))
  (shout [this] (str this "!!!")))

; extend strings
(extend-type java.lang.String
  Stringification
  (stringify [this] this)
  (shout [this] (str (clojure.string/upper-case this) "!!!")))

; handling vectors
(stringify ["a" 1 2]) ;-> "a12"
(shout [1 2 a]) ;-> "12A!!!"

; handing integers
(stringify 1) ;-> "1"
(shout 1) ;-> "1!!!"

; handling strings
(stringify "foo") ;-> "foo"
(shout "foo") ;-> "FOO!!!"
```

For the Tic Tac Toe, the idea was to create a `Player` protocol and extend the records `Human`, `EasyComputer` and `UnbeatableComputer`. It was not very practical, though. Protocols do not support functions with optional arguments, which would make the implementation very complicated: `select-spot` needs no argument if player is a `human`; it needs one argument (`board-length`) if player is an `easy-computer`; and it needs multiple arguments if player is an `unbeatable-computer`.

## Multimethods

The second option was, obviously, to have a `multimethod` that could handle optional arguments. The difference between them is that a `protocol` is a set of different methods, while `multimethod` is a single method.

It was not a simple task, and new questions kept coming up: Would the complexity of the arguments structure make it more difficult to implement and use a method? Should the namespaces and their functions be reorganized? How recursion handle optional arguments?

The first thing I tried was define the `multimethod` in one namespace and having the methods implementation in different ones. However, because I have been using alias to require functions from another namespaces, having methods in different files would defeat the purpose of a `multimethod`. I would still have to use conditionals: if player is a `human`, use `human/select-spot`; if player is `easy-computer`, use `easy/select-spot` and so on. Not very smart.

After reordering files and namespaces, I came up with this solution:

```clojure
(defrecord Player [marker role ai value])

(defmulti select-spot (fn [player & params] (:role player)))

(defmethod select-spot :human
  [player & params]
  (let [input (helpers/clean-string (read-line))]
    (if (helpers/is-int? input)
      (helpers/input-to-number input)
      (recur player params))))

(defmethod select-spot :easy-computer
  [player params]
  (rand-int (:board-length params)))

(defmethod select-spot :hard-computer
  [player params]
  (negamax/best-move (:board params)
                     (:current-player params)
                     (:opponent params)
                     (:depth params)))
```

**This post has an [update](http://pwdd.github.io/posts/multimethods-in-multiple-namespaces)**
