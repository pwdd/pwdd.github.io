---
layout: post
title:  "Hope You Guess my Name"
date:   2016-07-13 15:23:50 -0500
category: apprenticeship
tags: [clean-code]
---

I always liked learning. Seriously. I was that kind of a kid that liked going to school to watch the classes that other kids thought it was difficult. And I was good at that. But when I started reading code, I started to question my ability to learn. If it would take me so long to understand what the names of the variables meant, how much longer would it take to read a whole program? <!--more-->

One of the most upsetting things for me was to see things like this:

```javascript
while (t) {
  // keep doing something
}
```

I needed to go looking where that `t` came from, and some times find that `t` had been declared as `var t;`. Just that. So I would have to keep looking to find out where `t` was set to some value. After I found it, I would go back to the statement to see if it made sense. It was so bothersome that I thought that I would never be able to understand (any) code.

When I came across Ruby, things started to get a little better. Apparently, it is a *convention* to give readable names to everything in a Ruby program. But what about all the other languages? What is their *convention*?

## "Hello, dangerous world"

I started learning how to code by myself. I had a computer, an internet connection, a lot of passion and no money. Basically, the living translation of the word *naïf*. And one of the advices I found was: read a lot of code and learn from it. But reading code without a having an idea of what good code looks like is almost like reading *The Onion* without knowing that it is a satirical news website; or, even worse, reading *The Economist* without knowing that the world is bigger than Wall Street.

I know nothing about clean code. I just started my journey to learn more about it. One of the many reasons why I wanted so much to be an apprentice at [8th Light](http://8thlight.com) was the importance they give to well-written code. Since I started the program, I have been reading books about it, recommended by my mentors. I recently started [*Clean Code &mdash; A Handbook of Agile Software Craftsmanship*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), which made me realize how dangerous it was to read code without knowing what the best practices are.

The second chapter of the book is about giving good names to everything in a program. Can you imagine that? A whole chapter (almost 20 pages!) only to talk about the importance of good names? Now that the Tic Tac Toe project has grown, I can.

Just a few benefits from having well-named things:

* It's easier to understand what things are and what they do
* It's easier to search for a function or a class in the code base
* You don't feel stupid when talking with someone else about your code (just think about explaining how `t` behaves, and then how `remaining_time_in_seconds` behaves...)
* It makes comments unnecessary
* It makes it easier for others to read &mdash; and for ourselves to read it after a while

## Critical reading

The examples used in the book are written in Java &mdash; meaning, using readable names is not *a Ruby convention*, it is the best practice for every language. And that also means that the advice to *read a lot of code and learn from it* is actually a good one. It helps learn how to do it &mdash; and how not to. I have never thought it would be possible, but lately I have been reading code just like I read the news.
