---
layout: post
title:  "Undoing Changes with Git"
date:   2016-06-24 11:48:58 -0700
category: apprenticeship
tags: [git]
---

There are tons of articles out there trying to help people to make the best use of the tools they have available: "Google Tricks that Give You Better Search Results", "There Are More Things in Smartphones and Tablets Than Are Dreamt of in Your Philosophy"<!--more--> and other tips can be extremely helpful. Think about having the best bike that you could possibly buy. What a waste of money would it be if you didn't know how to use its gears? It would be just like any other bike.

The same happens with Git. It is an excellent tool to version control a project, but knowing only how to add and commit files does not exactly make anyone's life easier. Being able to check previous commits and revert changes are some of the things that make versioning control great.

## `checkout`, `revert`, `reset` and `stash`

While working on my Tic Tac Toe, I just wanted to throw away all changes I had done &mdash; none of them staged &mdash; and go back to my last commit. Usually, I would use `git checkout <filename>`. However, I had changed multiple files, and checking out file by file would be a tedious task. That is the moment when knowing all the *tricks* a tool can perform makes the life easier.

### `checkout`

When working with branches, `checkout` is used frequently to *check out* a branch:

- **Creating and checking out a branch:**

```shell
# at master branch
$ user@user-env ~/git_tests (master)

# create a branch (-b) named 'test-branch-one' and check it out
git checkout -b test-branch-one

# the branch was created and the HEAD is moved to 'test-branch-one'
$ user@user-env ~/git_tests (test-branch-one)
```


- **Moving from one branch to another:**

```shell
$ user@user-env ~/git_tests (test-branch-one)
git checkout master
$ user@user-env ~/git_tests (master)
```

What `checkout` does is to change `HEAD` from one place to another. A little deviation to learn about `HEAD`:

#### What goes on Git's `HEAD`?

`HEAD` is a pointer to the last commit of a branch that we are working on. So, if I am at `master` branch and I have three commits, `HEAD` will be pointing to the third one:

**master branch**

```
                     HEAD
------    ------    ------
| C1 |----| C2 |----| C3 |
------    ------    ------
```

`HEAD` is stored in a file inside `.git/HEAD`:

```shell
cat .git/HEAD
ref: refs/heads/master

cat .git/refs/heads/master
<third-commit-hash-number>
```

When `checkout` is used to move around branches, it changes where `HEAD` points to. If `test-branch-one` has two commits, `HEAD` will be on the second:

**test-branch-one**

```
             HEAD
-------    -------
| BC1 |----| BC2 |
-------    -------
```

```shell
git checkout test-branch-one

cat .git/HEAD
ref: refs/heads/test-branch-one
```

### Back to `checkout`

- **Checking out previous commits**

It is possible to `checkout` old commits with `git checkout <commit>`, which puts us in a *detached `HEAD`*. The good thing is that it is possible to go back to `HEAD` without harming the repository.

Considering a repository like the following:

```
git_tests
|_ .git
|_ first.txt
|_ second.txt
|_ third.txt
```

All the files inside `git_tests` directory are empty, an initial commit was made to track all of them, and we are in the `master` branch.

Make some changes to `first.txt`

**first.txt**

```
First commit
```

Then add to stage and commit:

```shell
git add first.txt
git commit -m "first commit"
```

Then make some more changes to `first.txt`:

**first.txt**

```
First commit

Second commit
```

Add and commit:

```shell
git add first.txt
git commit -m "second commit"
```

Take a look at the commits hashes:

```shell
git log

commit 239e...
Author: <author>
Date:   <date>

    second commit

commit 473d...
Author: <author>
Date:   <date>

    first commit
```

Checkout the first commit:

```shell
git checkout <first-commit-hash>

You are in 'detached HEAD' state. (...)
HEAD is now at 473d... second commit
```

Take a look at `first.txt`

**first.txt**

```
First commit
```

Make some changes from there, in the `first.txt` file:

**first.txt**

```
First commit

Changing file while in detached HEAD
```

Add and commit, then checkout back to `master`:

```shell
git add .
git commit -m "while in detached HEAD"
git checkout master
Warning: you are leaving 1 commit behind, not connected to
any of your branches:
```

**first.txt**

```
First commit

Second commit
```

Nothing has changed. And if we look at `git log`, we are going to see only the two commits we had originally:

```shell
git log

commit 239e...
Author: <author>
Date:   <date>

    second commit

commit 473d...
Author: <author>
Date:   <date>

    first commit
```

Where is the commit I did while I was in *detached `HEAD`*?

I could find it using `git reflog`:

```shell
git reflog

239e293 HEAD@{0}: checkout: moving from 473d... to master
(...)
26a739f HEAD@{3}: commit: while in detached head
(...)
```

That commit exists, but it does not belong in any branch and it did not change anything on the project.

- **Discard changes in a file**

With all of this back and forth around previous versions of the project, I think it is pretty clear that `checkout` does not actually change anything in a repository. It only change the place where `HEAD` is. That is why `git checkout <filename>` discards non-staged changes: it just goes back to `HEAD` (last commit) and, since there is nothing staged &mdash; and, consequently, committed &mdash; the changes in the specific file are lost. There is no going back.

The exception is the command `git checkout <commit> <filename>`. In this case, **only one file** will go back to previous state (to the `<commit>`). If changes are made to that file and then, staged and committed, those changes will reflect on `HEAD`.

Checkout an specific file in a previous commit

```shell
git checkout 473d... first.txt
```

Make changes to the file:

**first.txt**

```
First commit

Change this file when checking this out in a previous commit
```

Commit changes:

```shell
git add .
git commit -m "change one file from previous commit"
```

Now, when I run `git checkout master`, I get a message that I am already in master. The `git log` shows the last commit and `first.txt` did change.

After learning all those things about `checkout`, I thought I should be looking into some other option.

### `revert`

One of the greatest things about versioning control is to have access to the story of a repository. All the commits, the creation of the branches, merging, authors, date etc, it is all there. And `revert` maintains the story even when reverting to a previous commit. What it does, actually, is to create a new commit.

If I create a new file inside `git_tests`, add and commit it:

```shell
touch i-dont-want-this.txt

git add .
git commit -m "add new file to directory"
```

Then I decide that I don't care about this last commit and I just want to go back to the previous commit, I can use `revert`:

```shell
git revert <previous-commit-hash>
```

This will open the text editor so I can type the commit message:

```shell
Revert "change one file in previous commit"
```

After saving the message and quitting the editor, `i-dont-want-this.txt` is gone. However, it is part of the story of the repository:

```shell
commit 8cb...
Author: <author>
Date:   <date>

    Revert "change one file in previous commit"

commit a4c...
Author: <author>
Date:   <date>

    add new file to directory

# ... older commits
```

`revert` *reverted* the state of the repository to a previous commit, but it kept everything, including the changes introduced in the last commit.

### `reset`

While `revert` keeps the project story, `reset` does not. `reset` will **permanently** undo the changes. The command can be used in different ways:

- **Discarding staged changes**

  If I introduce some changes to the `first.txt` files:

  **first.txt**

  ```
  First commit

  Second commit

  I'm going to stage this
  ```

  Add changes to stage:

  ```shell
  git add first.txt
  ```

  And `reset` it back (unstage it)

  ```shell
  git reset
  ```

  If I open `first.txt`, I can still see the changes introduced earlier, but they are not staged anymore. If I try to commit that change, I get an alert message saying that there is a modified file that is not staged, so the commit cannot proceed:

  ```shell
  git commit -m "try to commit"

  (...)
  Changes not staged for commit:
          modified:   first.txt
  ```

- **Resetting to a previous commit**

  Another use for `reset` is to go back to a previous commit.

  ```shell
  git reset <commit>
  ```

  This will change the working directory to the given commit. **It will not discard any changes made past that point, but the changes are not going to be staged and all the newer commits will be gone**. `git log` will not show the newer commits anymore. The last one will be the one we are in.

  Even if I had the number of a newer commit, I could not go back using `git checkout <recent-commit>`.

### `stash`

When I found out about `stash`, I thought that it would be the best solution to my problem (undoing unstaged changes in multiple files), but it turned out to not be exactly what I wanted.

`stash` is used if we are working on a branch and want to move to another branch without having to commit any changes. This is possible with `git stash save`. It does not, however, throw away the changes. It just saves the unstaged changes to `stash`. Them, it would be necessary to run `git stash drop` to delete the changes from `stash`.

Again, it didn't look like the right choice for my problem.

### `checkout` was the solution...

After quite some time researching about how to undo changes in Git, `checkout` looked the best solution... So, considering that I am at `master` branch and I make changes in all files, without adding any of those files to stage, I could use `git checkout .` This would work exactly like `git checkout <filename>`, but for all files in the repository. It is a dangerous options if we are not sure that we have committed everything we need to before throwing everything away. In my specific case, it was the best option, though.
