---
layout: post
title:  "Where Is the Vector that Was Here?"
date:   2016-06-17 11:48:58 -0700
category: apprenticeship
tags: [clojure]
---

While building my Tic Tac Toe, I came across a situation like the following:<!--more-->

```clojure
(def board [1 2 3 4 5 6 7 8 9])

(get (map str board) 1)
;-> nil
```

The board was a vector, so I expected to get `"2"`, because

```clojure
(get board 1)
;-> 2
```

Why `(get (map str board) 1)` was returning `nil`?

Because `map` &mdash; as well as many other functions that operate on collections &mdash; returns a lazy sequence that does not respond to `get`. From that, I learned that using `nth` is a better choice. It works the same way for vectors, lists and for lazy sequences.

Lazy sequences, however, have a slower lookup than vectors, because they are not indexed. So, why the functions keep returning them?

They are actually very efficient: lazy sequences are a partial view into a collection that is potentially infinite and they are only *realized* when needed. It means that the computation is going to be deferred until it is necessary.

It is possible, for instance, to have an infinite collection of even numbers.

```clojure
(def even-numbers
  (filter even? (range)))
```

`range`, without a delimitation, is an infinite collection of numbers starting from zero.

```clojure
(realized? even-numbers)
;-> false
```

Then, it is possible to take a look into that infinite sequence:

```clojure
(take 10 even-numbers)
;-> (0 2 4 6 8 10 12 14 16 18)

(realized? even-numbers)
;-> true
```

`take` also returns a lazy sequence.

When it is *realized*, it gets cached and the lookup gets faster:

```clojure
(time (last (take 100000 even-numbers)))
|"Elapsed time: 162.707342 msecs"
;-> 199998

(time (last (take 100000 even-numbers)))
|"Elapsed time: 49.216146 msecs"
;-> 199998
```

Because of its efficiency, lazy sequences are a preferable way to solve some problems.

In "Programming Clojure", the authors came up with this (cool) solution for the [Fibonacci numbers](https://en.wikipedia.org/wiki/Fibonacci_number):

```clojure
(defn fibs
  []
  (map first (iterate (fn [[a b]] [b (+ a b)]) [0N 1N])))
```

The anonymous function used in `iterate` takes the parameters `a` and `b` from a vector `[0 1]` and returns a vector containing `b` and the sum of `a` and `b`.

```clojure
(take 5 (iterate (fn [[a b]] [b (+ a b)]) [0N 1N]))
;-> ([0N 1N] [1N 1N] [1N 2N] [2N 3N] [3N 5N])
```

Then, `map` will call `first` on each of those vectors.

```clojure
(take 5 (fibs))
;-> (0N 1N 1N 2N 3N)
```

Another possible solution for the problem is to use recursion:

```clojure
(defn recursive-fibs
  [limit]
  (loop [a 0N
         b 1N
         n limit
       acc []]
    (if (zero? n)
      acc
      (recur b
             (+ a b)
             (dec n)
             (conj acc a)))))

(recursive-fibs 5)
;-> [0N 1N 1N 2N 3N 5N]
```

They are not too different, except that `fibs` is shorter and easier to understand, and when it comes to big numbers, it is faster than using `recursive-fibs`.

```clojure
(time (take 10000 (fibs)))
|"Elapsed time: 1.632409 msecs"
;-> (0N 1N 1N 2N 3N 5N 8N 13N 21N 34N ...)

(time (recursive-fibs 10000))
|"Elapsed time: 34.860019 msecs"
;-> [0N 1N 1N 2N 3N 5N 8N 13N 21N 34N ...]
```
