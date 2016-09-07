---
layout: post
title:  "When the bug is in a test"
date:   2016-09-06 19:50:32 -0500
category: apprenticeship
tags: [TDD]
---

Last week I spent some hours re-reading and testing manually the functions that handle the medium computer. One of the tests wouldn't pass, and it was hard to figure out why: the code was right, the test was wrong. <!--more-->

I think that one of the most difficult part of Test Driven Development is to know *where to start*. We have a project &mdash; build a Tic Tac Toe &mdash; and we know how it will be like, but which one could possibly be the first test? Should we start with the board? What a board can or should do? What if we start with player? But what a player can do without a board?

It does not matter much where to start, but how: it should start simple. The complexity comes after a while. After two or three tests in place, things start to flow. Write a test to fail, make it pass, refactor, add another test. In the beginning of a project, this process helps finding out the direction to take. For instance, if we decide to start by writing the `board` portion of the project, and we want to make sure that a new board starts *empty*. What is *empty*? Is a `[]` (empty vector)? Or is `[0 0 0 0 0 0 0 0 0]`? Or `[[0 1 2] [3 4 5] [6 7 8]]`?

After we decide how to represent an empty board &mdash; a collection containing `:_` (the representation of an empty spot) &mdash; it is easy to move forward with the tests. A new board **should** have length equal to 9 and have all elements equal to `:_`.

Things can get harder later on, when we get used to the project. Besides having to test more complex things, we might get in a trap caused by knowing too well. This happened to me twice during the development of the Tic Tac Toe. In one case, I didn't write tests to cover all the corner cases. I knew what I wanted and I didn't think about testing that specific case.

Finding out that I haven't test the corner cases were comparatively easy to solve. After a while, I was surprised by a bug and reproducing it showed me that my function was not handling that situation &mdash; that was not being tested either. I wrote the test, then fixed the function and done.

As I read in the *Clean Code*, there are those who write tests to pass, instead of making them fail. I guess that is exactly what I did and that was my problem. I tested what I wanted, I forgot that I should know what I didn't want.

The other situation took me a lot longer, because it was hard to spot why the test was not passing. No matter what I would do, the test would not pass. I was so sure that the error was in the code that I didn't even considered that I had wrote a wrong test.

The function supposed to return an random number representing an index in a empty combo. For instance, if row `[0 1 2]` had nothing but empty spots, so it would return one of those indexes. The test I had used a 4x4 board, and it should return one of `[4 5 6 7]`, but it was also returning `[3 6 9 12]`. After trying everything, I realized that it was actually right. The board passed to the function in the test had the second row and the backward diagonal empty.

In this situation, there is no test for ourselves and our blind confidence that we wrote the test right. But it was a learning experience. I don't think I'll forget to check the test next time I come across one of those tests that simply do not pass. 
