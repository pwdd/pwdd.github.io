---
layout: post
title:  "Clojure thread macros"
date:   2016-09-14 13:13:32 -0500
category: apprenticeship
tags: [clojure]
---

The thread macros in Clojure are an excellent way to make things clearer and easier to read, but it can be hard to understand what they do and when to use them. <!--more-->

First things first: *thread* in this case has nothing to do with [threads of execution](https://en.wikipedia.org/wiki/Thread_(computing)). Here, it just means that we are passing a value to functions organized in a sequential order.

Considering this expression, `(- 2 (+ 1 (* 2 2)))`, this is how we could solve it step by step:

```
(- 2 (+ 1 (* 2 2)))
(- 2 (+ 1 4))
(- 2 5)
-3
```

We just solve the expression in the inner most parenthesis and passed it as a value to the next one and so for.

## Thread last

We can use the thread last macro `->>` to write the same form above in a nicer way. This thread will get a value and pass it as the **last** argument to the functions. One trick is to use three commas (`,,,`) in the place where the value would have been &mdash; and it would still be a valid form in Clojure, since commas are considered white spaces.

```clojure
; original expression
(- 2 (+ 1 (* 2 2))) ;-> -3

; thread last macro
(->> (* 2 2) (+ 1) (- 2)) ;-> -3

; thread last macro with visual representation
(->> (* 2 2) (+ 1 ,,,) (- 2 ,,,)) ;-> -3
; (- 2 (+ 1 4))
; (- 2 5)
; -3
```

## Thread first

The thread first macro `->` pass the value as the **first** argument to the functions. If, instead of the thread last we had used the thread first in the example above, we would have got 3 as the final result:

```clojure
; thread first macro
(-> (* 2 2) (+ 1) (- 2)) ;-> 3

; thread first macro with visual representation
(-> (* 2 2) (+ ,,, 1) (- ,,, 2))
; (- (+ 4 1) 2)
; (- 5 2)
; 3

; expanded form
(macroexpand '(-> (* 2 2) (+ 1) (- 2)))
;-> (- (+ (* 2 2) 1) 2)
```

## When to use them

They are used to replace nested forms, so reading becomes easier. As an example, considering the `list-all-files`, that is a private function that calls three other functions and return a collection of all names of the files inside a directory. It takes a directory, pass it to `files`, that returns all files objects that are then passed to `filenames`, that then passes all filenames with extension to `names`, that then returns a collection of the names of the files without extension.

```clojure
(defn files
  [directory]
  (filter #(.isFile %) (file-seq directory)))

(defn filenames
  [files]
  (map #(.getName %) files))

(defn names
  [filenames]
  (map #(subs % 0 (.indexOf % ".")) filenames))

(defn- list-all-files
  [directory]
  (names (filenames (files directory))))
```

It is a lot easier to understand using a thread macro. In this case, the thread first will do the job (although the thread last could also be used in this case, since the functions take only one argument):

```clojure
(defn- list-all-files
  [directory]
  (-> directory files filenames names)
```

One thing to notice is that when the function takes only one argument, it is not necessary to use parenthesis around it, just like the example above. 
