---
layout: post
title:  "\"S\", from SOLID"
date:   2016-08-08 19:50:35 -0500
category: apprenticeship
tags: [design-principles]
---

The popular wisdom says that "He who chases two rabbits will catch neither", or "Don't spread yourself too thin". The meaning of such sayings is that it is difficult to do more than one thing at a time. Just like people, classes, modules and functions are also not good at multitasking.

Considering this function:

```clojure
; violating SRP
(defn move
  [board position marker]
  (if (is-valid? position)
    (assoc board position marker)
    false))
```

It is doing more than one thing and violating the [Single Responsibility Principle (SRP)](https://en.wikipedia.org/wiki/Single_responsibility_principle) &mdash; the *S* from [*SOLID*](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)).

The SRP states that a function, a class or a module must have only one reason to change &mdash; which means it should have just one responsibility. In a [great article](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)) about it, Robert Martin (who coined the name of the principle) compares it to a business logic: in a company, the COO, CTO and CFO are all in the same hierarchic level, responding to the CEO; and we don't want to get the COO fired because of a change requested by the CTO. Both COO and CTO should be responsible only for the things related to their department.

In that example, `move` is doing too much. It

1. checks if `position` is valid,
2. associates `marker` in the `board`, and
3. returns `false` for some mysterious reason, making it difficult to get the *purpose* of the function.

Guessing what will be the reason that will make an application change is not a job for developers. It could be because `is-valid?` will take the `board` as an argument; or that `board` will become a `record` and `assoc` will not work anymore; or that `false` will be `true`.

This guesses can go on forever. So, instead of trying to make guesses, we would ideally organize things in a way that makes changes easier. That is when the design principles come in: they are guidelines that help us be ready for changes.

## How much is one?

We know if **a function has only one reason to change** by making sure that the function does only one thing. But, **what exactly is just one thing?**

At first, it seems reasonable that making a move only happens after checking if the `position` is valid. But, doesn't it seems more reasonable that `move` accepts only valid arguments? Why are we not checking if `board` or `marker` are valid?

Right now, getting input from user would be something like this:

```clojure
; a messy way to get user input
(defn get-input
  []
  (let [input (read-line)]
    (if (move board input marker)
      (move board input marker)
      (recur))))
```

It would be much simpler like this:

```clojure
(defn get-input
  []
  (let [input (read-line)]
    (if (is-valid? input)
      input
      (recur))))
```

We can then pass the returned value from `get-input` to `move`, that could be re-written like this:

```clojure
(defn move
  [board position marker]
  (assoc board position marker))
```

Now it is a function that does only one thing: associates a `marker` to a `board` in a given `position`. A good trick is to try to explain what a function does in just one sentence, without using words like *and* and *or*.

## The benefits of SRP diet

**Writing tests** for the second `move` example would be a lot simpler than trying to test the possible cases for the first `move` example. Instead of checking if `move` can take an invalid argument, and if it makes a move, and if it returns false, we can instead check only if it associates a value to the `board`.

Having a function doing only one thing reduces the risk of a change to have consequences in multiple points in the program. It also makes things more **reasonable**: a small change does not turn into a bigger job than it should be.
