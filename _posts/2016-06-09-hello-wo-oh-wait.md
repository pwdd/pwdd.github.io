---
layout: post
title:  "HELLO, WORL... Oh, wait! Where am I?"
date:   2016-06-09 10:33:58 -0700
category: apprenticeship
tags: [clojure, TDD]
---

As soon as I started working as a reporter some years ago, I heard about a guy who used to work at the same newspaper. He was famous for the things he wouldn't do.
<!--more--> 

It didn't take long for me to get my first advice: "Don't be like [name-of-the-guy]!" Soon I learned why: he would not pitch stories during the meetings, he would not deliver his work on time, and every time someone would ask him to do an interview, his answer would be, "They will not talk to me".

I'm not sure if that guy was real or some made up story so people would not behave like that. Because of it, you would never say you couldn't do something. Even if you had to work from midnight to 4:00 AM, without a place to sit, without water, waiting to do your interviews so you could be at the newsroom at 9:00 AM to write your article (yes, that was my first assignment as a reporter.)

## How to fail in the first week

I don't work as a journalist anymore. I'm now a (proud) student apprentice at [8th Light](http://8thlight.com). My first assignment: learn Clojure and start building a Tic Tac Toe. Maybe because of the "don't be like [name-of-the-guy]" advice, I set an ambitious deadline: I can do it, and quickly. 

Of course, I was not able to learn Clojure in 2 days and implement a Tic Tac Toe in 3. And I was the one who set an irrealistic deadline. "Fail" is not exactly what happened, although it was frustrating not getting things done on my first week. Somehow, I think that my *success* was the decision to ask for more time. Talking to my mentors was helpful and, when the stress of having a short deadline was over, I had a more productive day. Lesson learned: if I have a problem, it is better to ask for help earlier, rather than letting the problem grow bigger. 

## Lost in the functional world

I was familiar with Lisp syntax, so all the the parentheses and the prefix notations were easy to understand. Things got complicated when I started to think how to implement a function that would 'hold the state' of the game. I'm still working on it, and I hope to have an answer soon. 

### Who needs another *elegant* language?

I don't believe anyone who says that I should learn a language because it is "*expressive and elegant*" (the biggest cliche in programming books). The same argument is used all the time, no matter if the language is Ruby, Python, Clojure and probably others. I'm not sure if someone has ever written that you should learn a language because its syntax is a cluttered mess. 

So, ignoring the fact that Clojure is just another *elegant language*, the coolest part about learning a new language is to find out more about why the language was created, what it capable of, how and where it can be used. Right now I'm reading "Programming Clojure" and looking for those answers. And I am fascinated that, with Clojure, I am basically writing lists to accomplish the same thing I would have accomplished using Ruby. My goal is not to be a *rubyist*, a *clojurist* or *any-ist*. It is to be a good and well-grounded software developer. 

## My own glossary

I started to keep track of some of the words/sentences that I come across. I hope to be more comfortable with some concepts while I take notes and review them. *Homoiconicity* is one of them.  

**homoiconicity:** *Code as data*. In Lisp (List Processors), everything is encoded as data structure, every statement, even those that are executed and return a value:

```clojure
(let [x 1] (+ x 2))
; => 3

; let x and + are symbols
; 1 and 2 are literal integers
; [x 1] is a vector
; (+ x 2) is a list
; and everything is a big list
```





