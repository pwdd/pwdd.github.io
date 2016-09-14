---
layout: post
title:  "Destructuring in Clojure"
date:   2016-09-14 14:57:32 -0500
category: apprenticeship
tags: [clojure]
---

If the [thread macros]({% post_url 2016-09-14-clojure-thread-macros %}) that I wrote about on the previous post are a nice way to make nested forms easier to read, destructuring is a technique that makes code easier to read by making binding a more intuitive process. <!--more-->

There are many ways to *destructure* data, and what it does is to allow us to bind values to names in a more concise way. Although it might sound complex, it is actually very simple.

## Associative destructuring

Taking this case as an example:

```clojure
(let [first-name (:first-name params)
      middle-name (:middle-name params)
      last-name (:last-name params)])
```

It creates four new variables and each one of them is a value from a map. It might make things a bit clearer &mdash; it avoids calling `(:first-name params)` every time we mention that value in the function &mdash; but it could be clearer:

```clojure
(let [{first-name :first-name
      middle-name :middle-name
      last-name :last-name} params])
```

The solution above does the same thing in a more concise way. It is called *associative destructuring*, that also allow us to use only a few keys from a map, or use default values in case the key does not exist.

```clojure
; adds a default value in case the key ':middle-name' does not exist
(let [{first-name :first-name
      middle-name :middle-name
      last-name :last-name
      :or {middle-name "no middle name"}} params])
```

We could make it even shorter by using `:keys`:

```clojure
(let [{:keys [first-name middle-name last-name]} params])

(let [{:keys [first-name middle-name last-name]
       :or {middle-name "no middle name"}} params])
```

Just as a side note, `:keys` is used when the map has keywords as keys. It is also possible to use `:strs`, if keys are strings, or `:syms`, if they are symbols.

## Sequential destructuring

In this case, the destructuring is made using sequential data structure, like vectors. So this:

```clojure
(def a-vector ["a" "b" "c"])

(let [a (first a-vector)
      b (second a-vector)
      c (last a-vector)])
```

could become this:

```clojure
(let [[a b c] a-vector])
```

We can also get only one element and still be able to use the collection:

```clojure
(let [[a :as all] a-vector]
  (println a)
  (println all))
; a
; [a b c]
```

If we had a nested vector, we also could use sequential destructuring:

```clojure
(def nested-vector [[1 3] [2 6]])

(let [[first-vec second-vec] nested-vector
      [a b] first-vec
      [c d] second-vec]
  (println "Odds are " a " and " b)
  (println "Evens are " c " and " d))
;-> Odds are 1 and 3
;-> Evens are 2 and 6
```

## When to use it

The idea is to use destructuring on parameters passed to functions so that we can organized things better. This way, instead of calling accessing an element from a collection every time it is used inside a function, the value would be bound to some variable.
