---
layout: post
title:  "\"S\" for SOLID"
date:   2016-08-08 19:50:35 -0500
category: apprenticeship
tags: [design-principles]
---

The popular wisdom says that "He who chases two rabbits will catch neither", or "Don't spread yourself too thin". Just like people, classes, modules and functions are also not good at multitasking.<!--more-->

Considering this function:

```clojure
; violating SRP
(defn move
  [board position marker]
  (if (is-valid? position)
    (assoc board position marker)
    false))
```

It is doing more than one thing and violating the [Single Responsibility Principle (SRP)](https://en.wikipedia.org/wiki/Single_responsibility_principle) &mdash; the *S* in  [*SOLID*](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)).

The SRP states that a function, a class or a module must have only one reason to change &mdash; which means it should have just one responsibility. Robert C. Martin, who elaborated the concept of SOLID design, [once explained](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) the SRP by comparing it to responsibilities of people in a company: the COO, CTO and CFO are all in the same hierarchic level and they respond to the CEO; and we don't want to get the COO fired because of a change requested by the CTO. Both COO and CTO should be only responsible for the things related to their area.

## How can a function do too much?

In the `move` example, the function:

1. checks if `position` is valid,
2. associates `marker` in the `board`, and
3. returns `false` otherwise.

That means: when change comes, bad things will happen...  

Guessing the dark future is not a job for developers. It is impossible to know what will be the reason that will make that function to change. It could be because `is-valid?` will change its logic; or that `board` will become a `record` and `assoc` will not work anymore; or that `false` will be `true`.

These guesses can go on forever. So, instead of trying to predict the future, we would ideally organize things in a way that makes it easier to make changes. That is when the design principles come in handy: they are guidelines that help us be ready for changes.

## How much is one?

We know if **a function has only one reason to change** by making sure that the function does only one thing. But, **what exactly is just one thing?**

At first, it seems reasonable that making a move only happens after checking if the `position` is valid. But, doesn't it seems more reasonable that `move` accepts only valid arguments? Why are we not checking if `board` or `marker` are valid?

If `move` changes for whatever reason, we will also have to change things that now depend on it. The cascade effect is what we try to avoid. Right now, getting input from user would be something like this:

```clojure
; a messy way to get user input
(defn get-input
  []
  (let [input (read-line)]
    (if (move board input marker)
      (move board input marker)
      (recur))))
```

It would be simpler like this:

```clojure
(defn get-input
  []
  (let [input (read-line)]
    (if (is-valid? input)
      input
      (recur))))
```

Is `get-input` doing more than one thing? Well, it is if we think that it gets the input and checks if it is valid. But it is not if we consider that `get-input` returns a valid input and only that. So we can then pass the returned value from `get-input` to `move`, that could be re-written like this:

```clojure
(defn move
  [board position marker]
  (assoc board position marker))
```

Now `move` is a function that does only one thing: associates a `marker` to a `board` in a given `position`. A good trick is to try to explain what a function does in just one sentence, without using words like *and* and *or*.

## The benefits of SRP diet

**It is easier to test** the second `move` example. Assuming that the arguments are valid, we just need to make sure that it does what it says it does: put a `marker` on the `board`.  If we try to test the first example, we would have to cover a lot more: check if `move` has an invalid `position`, if it makes a move, and if it returns `false`.

Another benefit is that having a function doing only one thing **reduces the risk of a change to have consequences in multiple places in the program**. It also makes things more **reasonable**: a small change does not turn into a bigger job than it should be.
