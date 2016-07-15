---
layout: post
title:  "Version Out of Control"
date:   2016-07-14 15:23:50 -0500
category: apprenticeship
tags: [git]
---

This week, when one of my mentors sat by my side to talk to me about Git, it was a bit embarrassing. He looked at the history of the commits of my project and there they were: bad commits messages. <!--more-->

It has been a while since I wanted to have a better workflow routine when making commits. I always forget to do it, then I do it in the wrong moment. So the messages don't say anything helpful, because the commit itself is not ideal. When this mess becomes public, it is far from helpful for those who are working in the same project.

## **What a good commit looks like**

A good commit is a working version of the project: all tests pass and the program works up to that point in time. Everyone should be able to `checkout` and older commit and still have everything working, even without newest features.

And a commit message has [all the information necessary](http://chris.beams.io/posts/git-commit/) to track what was done. It has a subject line and a message with a list of the most important things that were implemented.

To write the subject and the body message, don't use `git commit -m`, but `git commit -v`, so that a text editor will open.

## **Fixing older commits with `rebase`**

Commits do not need to be perfect all the time. It is possible to make *personal* commits. For instance, there are two failing tests in a program. One of them was fixed, so it is a good place to commit, before starting working on something else. Then, after the second test is passing, commit again. In order to remove the commit that is not good (the one make while we still had a failing test), we can use `git rebase`.

We can do something like:

```shell
git rebase -i head~3
```

The `-i` flag means *interactive*, so that we can decide what to do, and `head~3` will get the last three commits.

Then, we will see this:

```shell
pick 123456 an older commit
pick 234567 one test pass, one test fail
pick 345678 all test pass
```

The subject lines of that example are not good, but it is just for the sake of the example.

Just below that, we can see our [options to rewrite the history of the commits](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History):

```shell
Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
```

They are pretty much self-explanatory. In the above example, we could use `fixup`, that will *merge* the last commit into the previous (*up*) commit and discard the last commit message (if the message is important, it is better to use `squash`). All we have to do is to literally replace the word `pick` by `fixup` and save the file.

```shell
pick 123456 an older commit
pick 234567 one test pass, one test fail
fixup 345678 all test pass
```
