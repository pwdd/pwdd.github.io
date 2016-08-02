---
layout: post
title:  "Getting to Know Clojure"
date:   2016-08-01 18:25:12 -0500
category: apprenticeship
tags: [clojure]
---

It's been almost two months since I started learning Clojure and I think I'm finally letting it go my problem with the documentation &mdash; that is, for me, one of the most complicated things about the language. <!--more-->

This is the definition of `if-let`:

> bindings => binding-form test
>
> If test is true, evaluates then with binding-form bound to the value of test, if not, yields else

I have no clue of what that means and there is no solution for that. That's why I'm trying to let it go. Stop complaining and start trying it out. Here is the `if-let` *in action*:

## `if-let`

```clojure
; expression returns a truthy value, so 'x' is bound to that
(if-let [x (get [1 2 3] 0)]
  x
  "no index 0 on vector")
;-> 1

; expression returns 'nil', so x is not bound to any value and 'else' form is evaluated
(if-let [x (get [1 2 3] 3)]
  x
  "no index 3 on vector")
;-> no index 3 on vector

; expression returns 'nil' and, as there is no 'else', return 'nil'
(if-let [x (get [1 2 3] 3)]
  x)
;-> nil

;; Note that it does not mean that 'x' was bound to 'nil'.
;; If the expression evaluated to 'false', it would return 'nil',
;; like in the example below:

(if-let [x (contains? [1 2 3] 3)]
  x)
;-> nil
```

Other things that I learned by trying it out:

## `when-let`

This one is similar, but it uses the `when` logic (*when something is true, do this; there is no 'else'*)

```clojure
(when-let [x (get [1 2 3] 0)]
  (println x)
  (inc x))
;| "1"
;-> 2
```

## Multiple elements from a vector

### With `mapv`

```clojure
(mapv [:a :b :c :d :e] [1 3])
;-> [:b :d]
```

### With `map`

```clojure
(map [:a :b :c :d :e] [1 3])
;-> (:b :d)
```

`map` and `mapv` take a function and a collection. In this case, vector is the function and the argument is a collection of indexes.


```clojure
; the vector is the function and 1 is the argument
([:a :b :c :d :e] 1)
;-> :b
```

The same can be used to get multiple values from a `hashmap`, that is also a function:

```clojure
(mapv { :a 1 :b 2 :c 3 :d 4 :e 5 } [:a :c])
;-> [1 3]
```

## `map-indexed`

It *maps* an element to its index. The index is passed as the first argument of the function:

```clojure
(map-indexed vector [:a :b :c])
;-> ([0 :a] [1 :b] [2 :c])

(map-indexed + [0 10 100])
;-> (0 11 102)
```

## `keep-indexed`

The difference from `map-indexed` is that it gives more flexibility when passing arguments to its functions. Note: `%1` is index and `%2` is value:

```clojure
; if the index is even, return the value associated with it
(keep-indexed #(if (even? %1) %2) [:a :b :c :d :e])
;-> (:a :c :e)
```

***

One last thing: I have been using Clojure REPL a lot so I can try things out. Of course, it is easier to write on file and then load it into the REPL.

```clojure
(load-file "path/to/file")
```

After making changes to that given file, it is necessary to reload the REPL:

```clojure
(use 'namespace :reload-all)
; where 'namespace' is the one we are in, for instance, 'user'
; and ':reload-all' will reload the file and any other dependency
```
