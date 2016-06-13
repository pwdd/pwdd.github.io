---
layout: post
title:  "A Week With Clojure"
date:   2016-06-10 10:33:58 -0700
category: apprenticeship
tags: [clojure, TDD]
---

After a week using Clojure, I think I finally started to get used to reading it. Maybe because I'm not a native English speaker or because I am used to Ruby, I've been having a hard time with some conventions. <!--more-->While reading the docs, I see things like `coll` and `exprs`, and they make no sense. It took me a while to understand that they mean *collection* and *expressions*. Reading "Programming Clojure" has been helpful. Besides having a section about naming conventions, the examples used in the book are clearer than those in the docs.

## Comparing Apples and Oranges

I tried not to compare Clojure with Ruby because I think it is not very productive. First, one language is functional and the other is Object-Oriented. Second, each language has its own *idiosyncrasies*. Then I realized that it is difficult to avoid making comparisons.

When I was learning English, it was inevitable to compare a few expressions with those in Portuguese, even though they are completely different languages: English is West Germanic and Portuguese is a Romance language. It took a while till I naturally stopped comparing them. Things started to make sense by themselves. I didn't need to translate English into Portuguese in order to understand it.

At least for now, it is impossible for me not to compare Clojure and Ruby. I hope that in the future I won't need to keep doing it.

A few similarities I found between them:


- Everything is true, except `false` and `nil`.
  {% highlight clojure %}
  (if 0
    "Zero is true"
    "Zero is false")
  ;-> "Zero is true"

  (if ()
    "Empty list is true"
    "Empty list is false")
  ;-> "Empty list is true"

  (if -1
    "Negative one is true"
    "Negative one is false")
  ;-> "Negative one is true"
  {% endhighlight %}

- Predicate methods end with a `?`
  {% highlight clojure %}
  (nil? 0)
  ;-> false

  (integer? 0)
  ; -? true

  (string? 0)
  ; false
  {% endhighlight %}

  Talking about the predicate methods, `true?` and `false?` are tricky. They will return `true` if the *evaluated expression* is a Boolean. That means that, even if `"a"` is truthy, it will evaluate to the string `"a"` and `(true? "a")` will return `false`.

  {% highlight clojure %}
  (true? "a")
  ;-> false

  (true? (= 1 1))
  ;-> true

  (false? "a")
  ;-> false

  (false? (= 1 2))
  ;-> true
  {% endhighlight %}

- Print statements return `nil`

  {% highlight clojure %}
  (def t (println "I'm not t"))

  (nil? t)
  ;-> true
  {% endhighlight %}

- Everything is evaluated and a function returns the last statement.

  {% highlight clojure %}
  (defn adding-nothing [x, y]
    (+ x y)
    ":-)")

  (adding-nothing 1 2)
  ;-> ":-)"
  {% endhighlight %}

- There is usually more than one way to accomplish the same thing

  {% highlight clojure %}
  (def m {:a 1 :b 2 :c 3})

  ; a 'map' is a function and a key can be passed to it
  (m :a)
  ;-> 1

  ; a 'keyword' is also a function and a 'map' can be passed to it
  (:a m)
  ;-> 1

  (get m :a)
  ;-> 1
  {% endhighlight %}

## Vector or Lists?

What sense does it make to use vectors instead of lists, when you are basically writing lists? And yet, vectors are everywhere.

Vectors are indexed, so they provide a quicker look up, while lists provide linear access to its elements. That means that, if I want the nth element inside a vector, it will access only the nth element. A list would be sequentially scanned in order to find the nth element.

## Is This a List?

When I started throwing things on REPL to see how they get evaluated, I noticed that something different &mdash; at first, I thought the returned values were *lists*.

```clojure
(map str [1 2 3])
;-> ("1" "2" "3")

(map str '(1 2 3))
;-> ("1" "2" "3")

(take 2 [1 2 3])
;-> (1 2)

(take 2 '(1 2 3))
;-> (1 2)
```

My first reaction: force the vector to stay a vector:

```clojure
(vec (map str [1 2 3]))
;-> ["1" "2" "3"]
```

Of course, it makes no sense to do it all the time. Then I tried this:

```clojure
(= [1 2 3] '(1 2 3))
;-> true
```

It led me to learn more about sequences (that look like lists in REPL). Sequences (`seq`) are an abstraction and they must respond to `first` (returns its first element), `rest` (returns everything after the first element), and `cons` (*constructs* a new sequence).

That is why `map` (and other methods like `reduce`, `take`) can take a vector, or a list, without throwing a type error.

Maps and sets are also sequences, but, as they are unordered, performing functions like `rest` or `take` return unexpected sequences.

```clojure
; take 2 elements from a set
(take 2 #{1 2 3 4 5})
;-> (1 4)

(rest #{1 2 3 4 5})
;-> (4 3 2 5)
```

Still in the *comparing mood*, I've been trying to see what sequences and Ruby enumerators have in common, but I'm still not sure if they are comparable.

## It is Important to Maintain the Order

It is pretty clear that arguments passed to functions have a defined order. `(get 1 [1 2 3])` won't work.

```clojure
; There is no `[1 2 3]` in `1`
(get 1 [1 2 3])
;-> nil

; the right order
(get [1 2 3] 1)
;-> 2
```

The order of things matter even when it is not obvious, like checking if something is equal to another:

```clojure
(= 1 1)
;-> true

(= 2 1)
;-> false

(= 1 2)
;-> false
```

It matters specially when testing functions. I'm working [Speclj](https://github.com/slagyr/speclj) to test my Tic Tac Toe and sometimes I get some cryptic messages:

```clojure
; wrong implementation of a function
(defn test-f
  []
  "this is NOT the expected result")

; test with wrong order of arguments
(describe "test-f"
  (it "test order of arguments"
    (should= (test-f) "this is the expected result test")))

; the error message
;-> Expected: "this is NOT the expected result"
;->      got: "this is the expected result test" (using =)
```

In the example, it is pretty obvious what is wrong, but when some more complex function is being tested, a message like that is misleading and unhelpful, specially if the test acts as the documentation of the function.

```clojure
; the correct order
(describe "test-f"
  (it "test order of arguments"
    (should= "this is the expected result test" (test-f))))

; error message
;-> Expected: "this is the expected result test"
;->      got: "this is NOT the expected result" (using =)
```
