---
layout: post
title:  "Programming Exercises Endorphins"
date:   2016-07-20 15:07:39 -0500
category: apprenticeship
tags: [clojure, TDD]
---

Physical exercises can reduce stress, ease anxiety and depression, and boost self-esteem. Programming exercises can do the same. <!--more-->

Before I move on: no, there is no new study saying that coding exercises have the same benefits as physical activities; there is no scientific evidence that support that statement.

But what a great thing is to work on a code kata!

One of the stories that I have been working on this week is the [Prime Factors](https://en.wikipedia.org/wiki/Prime_factor) kata. The idea of a kata is to solve a problem, delete it all, and do it again, till you have it memorized, and you can solve the problem in 5 to 10 minutes.

## TDD

The first thing to do when working on a kata is to write a test. The idea is to follow the [Three Rules of TDD](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd). They are:

1. Don't write code until you have written a failing unit test
2. Don't write more of a unit test that it is sufficient to fail
3. Don't write more code than is sufficient to pass the currently failing test

Following the rules was actually one of the things that made the exercise so great. Usually, when I have to solve a problem, I tend to try to find the final solution immediately. And, of course, it is stressful. The idea that I *must* come up with the final solution puts an unnecessary weight on writing code and tests. Taking small steps is a lot easier.

The first thing to do is to write a test to fail:

```clojure
(describe "prime-factors-of"
  (it "returns an empty collection if number is smaller than 2"
    (should= [] (prime-factors-of 1))))
```

The test will fail. We then write the minimal amount of code so that the test can pass:

```clojure
(defn prime-factors-of
  [number]
  [])
```

The test pass.

It might seem like a waste of time to write something like that. After all, everybody knows that the solution will never work after we add one more simple test. But that is not the point. Taking a small step at a time makes it easier to understand the problem as a whole, and it makes sure that we are not going to have untested code. And, it makes it feel like we are making progress, instead of being stuck.

Next, time to write one more failing test:

```clojure
(describe "prime-factors-of"
  (it "returns an empty collection if number is smaller than 2"
    (should= [] (prime-factors-of 1)))
  (it "returns [2] if number is 2"
    (should= [2] (prime-factors-of 2))))
```

Test fails, so we need to fix the code:

```clojure
(defn prime-factors-of
  [number]
  (if (< number 2)
    []
    [number]))
```

The test pass, and the next step is to write another failing test:

```clojure
(describe "prime-factors-of"
  (it "returns an empty collection if number is smaller than 2"
    (should= [] (prime-factors-of 1)))
  (it "returns [2] if number is 2"
    (should= [2] (prime-factors-of 2)))
  (it "returns [2 2] if number is 4"
    (should= [2 2] (prime-factors-of 4))))
```

It will fail, and we must keep moving forward. When everything is done &mdash; all tests are in place and all of them pass &mdash;, we delete it all and start over.

It looks like an exhausting process, but it is not. I mean, it is tiring, but it is just like physical activity. It makes you feel good in the end. It is somehow a reminder of why coding is so fascinating: the simple joy of solving a problem and learning new things while you do it.

## The argument against code katas

I once read a good article written by a developer that thinks that code katas are useless. They believe that doing the same thing over and over again, instead of looking for better solutions, does not make anyone a better programmer. They compared katas to walking: walking everyday does not make anyone a *professional walker*. In order to be a better walker, it is necessary to add obstacles to a routine, so one can improve their walking skills.

The developer might have a point. I am not experienced enough to say how much performing katas will be helpful in the future. I know that practicing something the wrong way can perpetuate bad habits. I would, however, argue that no one works on kata to be a *professional kata solver*. Just to change the metaphor a little bit: pianists, even those that are [among the greatest of them](https://www.youtube.com/watch?v=rpdoE7iWbJk), keep the habit of practicing. Playing scales or warm up sequences by heart does not mean that they are good only at that.

The comparisons are good as a metaphors, but coding is not like playing an instrument or exercising. Katas might be helpful to some people and useless for others. After all, each one has their own way of learning. I learned a lot about TDD and `loop/recur` in Clojure while working on the prime factors problem. And, well, I intend to keep practicing katas. If not to improve my skills, at least to have fun.

## The side effect of a kata

This is the 7th week of my apprenticeship and I have been struggling to get my tests running automatically since the first day. Running [`lein spec -a`](https://github.com/slagyr/speclj/#autotest) was not helpful. More than that, it was actually very troublesome and caused me to spend a lot more time than I had to.

Instead of having the tests running automatically when I saved a file, I would have to restart the process every time. And I only realized that I had to do it after spending hours trying to understand why I could not fix an error. The fact it that that error had been fixed, but the tests were not being updated. Sometimes, I thought things were passing and they were not; and sometimes I thought they were failing for some reason, when the error was already something else. It was extremely frustrating.

Then, when I started practicing the kata, I wanted something that would make it easier for me to have access to the console (since I would be restarting the tests a lot!). I decided to try [PlatformIO-IDE-Terminal](https://atom.io/packages/platformio-ide-terminal), a plugin for [Atom](https://atom.io/). For my surprise, running `lein spec -a` in the instance of the terminal inside Atom actually keeps the tests running automatically! It updates everything perfectly. And it would probably have saved 30% of my time if I knew about it when developing the Tic Tac Toe.
