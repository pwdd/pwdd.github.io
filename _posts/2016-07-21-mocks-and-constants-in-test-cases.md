---
layout: post
title:  "Mocks and Constants in Test Cases"
date:   2016-07-21 16:38:20 -0500
category: apprenticeship
tags: [clojure, TDD]
---

One of the requirements of the Tic Tac Toe project was that the screen should be cleared after getting an input from the user. Everything on screen should disappear, except the representation of the current state of the board. The problem was that my tests started to disappear as well. <!--more-->

I have been using [Speclj](https://github.com/slagyr/speclj) as my test framework and, every time I ran `lein spec -a`, I would get big blank spaces in the middle of the test messages. Something similar to this:

```shell
get-user-attributes




- returns a map that contains the key :marker
```

At first, I thought that the problem was with `clear-screen` being called during the tests of every function that uses `prompt` (that class `clear-screen`). I even found a way to mock it:

```clojure
(describe "prompt")
  (around [it]
      (with-redefs [clear-screen (fn [])]))
  ; ...
```

`around` will be responsible for invoking the function *around* each `it` block inside the `describe`. And `with-redefs` is a Clojure macro that is used to temporarily redefine a function.

The big empty spaces were gone, but I was still getting some weird behavior when running the tests.

## Using constants

Here is an example of what was going on:

```clojure
; the tests
(describe "is-board-full?"
  (let [_ empty-spot
        empty-board (new-board)]
  (it "returns false if board is empty"
    (should-not (is-board-full? empty-board)))
  (it "returns true if board is full"
    (should (is-board-full? (vec (repeat board-length :x)))))
  (it "returns false if there is any spot available"
    (should-not (is-board-full? [:x :o :x :o _ :x :x :o])))))
```

And here is the output of the above successful tests:

```shell
is-board-full?
  returns false if there is any spot available
```

Why were the first two tests disappearing?

The problem was the use of `let`. In Clojure, the last statement is *returned*. So, although the two first tests were running, only the message for the last test would be visible.

In this case, `with-redefs` would not be appropriate, because my intention was not to mock anything, but only create some constants that would be used in all the `it` blocks.

The solution was to use `with`, that declares a symbol lazily evaluated once per block.

```clojure
(describe "is-board-full?"
  (with _ empty-spot)
  (with empty-board (new-board))
  (it "returns false if board is empty"
    (should-not (is-board-full? @empty-board)))
  (it "returns true if board is full"
    (should (is-board-full? (vec (repeat board-length :x)))))
  (it "returns false if there is any spot available"
    (should-not (is-board-full? [:x :o :x :o @_ :x :x :o]))))
```

Changing that also updated the total of tests that were run (the number is shown in the end of the test messages). It went from 123 to 191.

## The unnecessary mock

After that, I went back to review the mocks, and they didn't seem right. For instance, in the situation in which I mocked `prompt`, in order to avoid the blank spaces, the tests that called `prompt` recursively were bugged. They would pass even with wrong inputs.

The solution for that was to use `around` to stub the string outputted by `prompt`.

```clojure
(describe "prompt"
  (around [it]
    (with-out-str (it))))
  ; ...
```

In this case, `with-out-str` stub the output, **including the escape sequence used to clear the screen**.

Inside the tests that call `prompt`, I did the same and the recursive tests were fixed:

```clojure
(describe "get-marker"
  (around [it]
    (with-out-str (it)))
  (it "returns a player's marker if input is valid"
    (should= "x" (with-in-str "x" (get-marker))))
  (it "recurs and keep asking for input until it is valid"
    (should= "x" (with-in-str "1\n#\n x" (get-marker))))
  (it "recurs and asks for new input if marker is being used by the first player"
    (should= "o" (with-in-str "x\no"
                   (get-marker)))))
```
