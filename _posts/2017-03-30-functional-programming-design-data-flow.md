---
layout: post
title:  "Functional Programming Design - Data Flow"
date:   2017-03-30 12:50:35 -0500
category: apprenticeship
tags: [scala]
---

One of the arguments used to praise functional programming is that "*it is easy to reason about*". In a simplistic sense, it is in fact easy to think that something takes an input and returns an output. But there is much more to that argument.<!--more-->

The idea of *reasoning about* comes from the fact that FP follows some laws (just like Math) and, if we are working with referentially transparent data and/or functions, most of the times we are able to prove those laws. For instance, we can say that, in any point in time, the left part of an operation is equal to the right part.

```scala
def add(x: Int, y: Int) = x + y

add(add(20, 20), add(-3, 5)) = 42
// add(20 + 20,     -3 + 5)  = 42
// add(   40     ,    2   )  = 42
```

In the example above, we can get the answer `42` in any point we start the evaluation. There is no state being changed along the way to make the final result to be different.

## Reasoning about data

One might argue that it is easier to understand the world in an imperative way (*sequentially: do this, then do that*). But it is not that more difficult to think about the way the data is used by *this* and *that*. In a [brilliant article](http://www.lihaoyi.com/post/WhatsFunctionalProgrammingAllAbout.html) about functional programming, [Li Haoyi](http://www.lihaoyi.com/Resume/) uses a tiramisu recipe to show the difference between imperative and functional programming.

The idea is that we should think about separate the steps of a program in a way that:

1. Different parts of the program can be executed in parallel
1. The dependencies are obvious (the parts of the recipe that need to occur before or after some step)

We need to think about what we can prepare first, what we can prepare in parallel, what we have to prepare sequentially. This way, everything can happen independently and, when the dependency happens, we have a clear idea of what is going on at each step of the process.

By doing this, we avoid errors in our program. The dependencies are clear and we know that changing the order in which the execution happens will throw an error (something needed to a given step has not yet been evaluated), while in imperative way we would have to investigate the changes in the state of something.
