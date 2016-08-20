---
layout: post
title:  "The difference between merge and rebase"
date:   2016-08-19 15:16:06 -0500
category: apprenticeship
tags: [git]
---

Why would you rebase if you can merge without it? To keep things organized. <!--more-->

First things first:

## What is a **fast-forward** merge?

We [know]({% post_url 20116-06-24-undoing-changes-with-git#checkout-revert-reset-and-stash %}) that `HEAD` is a pointer to the most recent commit of the branch we are in. If we are currently in `master`, and it has 3 commits, this is how it would look like:

```
currently checking out master
                            ------
                           /| C4 | FEATURE
                     HEAD / ------
------    ------    ------
| C1 |----| C2 |----| C3 | MASTER
------    ------    ------
```

If we merge the branch `feature` into `master`, as there is no conflict, we can do a fast-forward merge:

```
HEAD simply moves forward, from C3 to C4


                       >........HEAD
------    ------    ------     ------
| C1 |----| C2 |----| C3 | ----| C4 |
------    ------    ------     ------
```

## `merge`

A more common situation when merging usually is not the simple. It can happen that `master` moves ahead before `feature` can merge. Then we have:

```
master has moved ahead before feature was merged
                            ------
                           /| C4 | FEATURE
                     HEAD / ------
------    ------    ------    ------
| C1 |----| C2 |----| C3 |----| C5 | MASTER
------    ------    ------    ------
```

If we merge with `feature` into `master` (`git merge feature`), this is what happens:

```
After 'git merge feature' from master, this is how master looks like:
                              ------
                           /--| C4 |\
                          /   ------ \   HEAD
------    ------    ------    ------  \ ------
| C1 |----| C2 |----| C3 |----| C5 |----| C6 | MASTER
------    ------    ------    ------    ------
```

We did a **three way merge**: we solved the conflict and created a new commit (`C6`) in order to merge `feature` into `master`.

## `rebase`

Rebasing before merging is usually the best solution when dealing with the above situation. It will *unplug* the `feature` from `C3` and make it into a **new commit on top of `master`**.

From `feature`, we run `git rebase master`

```
Rebase feature on top of master with 'git rebase master'
                                      -------
                                   /--| C4' | FEATURE
                                  /   -------  
------    ------    ------    ------    
| C1 |----| C2 |----| C3 |----| C5 | MASTER
------    ------    ------    ------
```

Now, when we merge `feature` into `master`, it is just a **fast-forward** merge:

```
A fast-forward merge after rebasing

                                  >........HEAD
------    ------    ------     ------    -------
| C1 |----| C2 |----| C3 | ----| C5 |----| C4' |
------    ------    ------     ------    -------
```

It looks like the new commit didn't come from another branch, as if everything was done on `master`, since there is no deviation like what happens with a three way merge. But, of course, all the information is kept in the commits. So, if it happens that we need to know who is the author of a given commit, it will be there.  

## `rebase` with a remote

Assuming that our work is in a remote, it is good to keep the branches we are working on updated, so the process of rebasing and merging has less conflicts. We can do a `git pull --rebase` so that it will `pull` the changes from remote but, instead of automatically merging, it will do a `rebase` first.

## Why rebasing?

There are a few reasons we would like to keep things clean. One of them is that it keeps the history clean and understandable. If we consider [a commit to be like a chapter of a book]({% post_url 2016-08-17-how-a-good-commit-looks-like %}), we have a clean and linear history to tell.

The most important reason is that it makes it easier to collaborate. If we are working on a project, or want to collaborate in one, we can rebase before making a pull request, so that the maintainer of the code base does not need to be dealing with conflicts and can, instead, do a clean fast-forward merge.

## See how history looks like:

If we want to check it out how the history of our project looks like, we can access a ANSI graph provided by Git. It will be just like the ones in this page, put vertically displayed:

```shell
git log --oneline --decorate --graph
```
