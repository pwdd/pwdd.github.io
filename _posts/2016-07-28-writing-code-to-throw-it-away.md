---
layout: post
title:  "Writing Code to Be Thrown Away"
date:   2016-07-28 20:49:36 -0500
category: apprenticeship
tags: [TDD]
---

One of my recent assignments was to pair with another apprentice in the Game of Life. What was weird at first was the requirement to throw the code away after the game was done. <!--more--> The idea was to build it, delete it, build it again in another language, always following the [Three Rules of TDD](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd).

There were many great things about that experience. One of them was the chance to work with someone in a project starting from scratch, always discussing the best and/or easier way to implement some things. Another one was exactly the fact that I could throw code away.

It was not the first time that *throwing code away* came up. One of the advices I got was that, when stuck, sometimes the best thing to do is to delete it and start over. Recently I had to *throw away* parts of the Tic Tac Toe and rebuild it using the Three Rules of TDD (not really deleting it, but ignoring the implementation that already existed).

Back to the pairing assignment: writing code that would not last was liberating. It might seem contradictory, since I was working on something that was being written to be thrown away, but it makes sense &mdash; at least to me. Approaching a problem without considering external influences (Who's gonna see it? Does this code shows how much I know about this project? Is it flawless?) puts the focus only on the problem in front of us and nothing else.

It also makes it easier to follow the Three Rules of TDD. If we are not thinking about the best possible solution for a given problem, it is easier to write the simplest solution to make the test pass. It does not mean that we should have a license to write bad code. Quite the opposite: refactoring is part of the routine of TDD.

## Resisting the opportunity to throw code away

At the point that I am now in the Tic Tac Toe project, sometimes I wish I could throw everything away and start it over, with the knowledge that I have now and I didn't have before. That is what I'm trying not to do.

I need to get better at refactoring, and in this case, deleting code is not helpful. One of the things that I need to work on is on my timing: when to refactor? Immediately after the tests pass? Later, when there is more information about how those functions are going to be used elsewhere? What if that information never comes?

That is why I'm resisting the wish to start everything over. I need to practice refactoring, and it has not been an easy task. It is frustrating to look at something, know that it needs change and, at every try to make it better, you fail miserably. But I must say that I have made some progress. And, because it is so hard and so slow, when you make the smallest progress, that feels great.
