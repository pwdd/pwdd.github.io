---
layout: post
title:  "What a good commit looks like"
date:   2016-08-17 18:16:06 -0500
category: apprenticeship
tags: [git]
---

If you want to collaborate with a project, would you rather branch from the commit "animation works" or from "Add animation to homepage". Which gives you the idea of a reliable point of the project? If the developer took care of writing a good commit message, can we trust that that commit is a good place to start working? <!--more-->

First, how a commit is made:

```
A 'Initial commit' of a repo with one file, 'readme.md'

|-------------------|          
|    COMMIT 1a2s3   |         |----------------------|           |--------------|
|      tree 1234e   |         | TREE 1234e           |           | BLOB 1w2e3   |
|    author <name>  |=======>>| blob 1w2e3 readme.md |=========>>| file content |
| committer <name>  |         |----------------------|           |--------------|
| "Initial commit"  |
|-------------------|
```

If we were tracking more files, each one would have a **blob**, and they would be listed in the **tree**. We can say that the tree and the blob make up the snapshot of our project at that point in time. When we make the second commit, this is what happens:

```
Second commit points to its parent and each commit points to its own snapshot

         C1                                 C2
|-------------------|              |-------------------|          
|    COMMIT 1a2s3   |              |    COMMIT 5h6j7   |
|      tree 1234e   |              |      tree 1B3Xe   |
|    parent         |              |    parent 1a2s3   |
|    author <name>  |<<============|    author <name>  |
| committer <name>  |              | committer <name>  |
| "Initial commit"  |              | "SECOND commit"   |
|-------------------|              |-------------------|
         ||                                ||
    |-----------|                     |-----------|                   
    | snapshot  |                     | snapshot  |
    |-----------|                     |-----------|
```

## Good message

A commit message should give enough information about what was added to the project with that commit. There are a few good articles out there about the rules of writing messages ([here](http://chris.beams.io/posts/git-commit/) and [here](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)).

In short: ideally, a commit message has a capitalized explanatory title, using imperative verb tense, and a body containing **what** has changed and **why**.

When commiting, instead of using `git commit -m <title>`, a good idea is to use `git commit -v`, that will bring up the text editor and show `git diff` (the difference between the previous commit and the one that is being made).

## Clean and reversible

We can think of a commit as a chapter of a book about our project. We can read a book up to certain point and hopefully understand everything up to that point. We would not find mistakes or notes from the author saying that they will fix something later. It should be the same with a commit.

Let's say that we are working on a console game and we have to add colors to its output. Another developer is interested in the game, but they don't want it to be colorful. They should be able to branch from the commit before we started adding colors, and they should have a version of the game that should work perfectly up to that point, and with all tests passing.

Having a reversible commit is also useful when debugging: if everything worked alright in the previous commit, we can simply compare what has changed from that point on. Or go back to that point and discard the most recent changes. Reversible commits build us a safe net for when we need it.

*<next post is about maintaining Git history clean>*
