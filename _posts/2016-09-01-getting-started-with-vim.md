---
layout: post
title:  "Getting started with Vim"
date:   2016-09-01 09:27:41 -0500
category: apprenticeship
tags: [vim]
---

I tried to use [Vim](http://www.vim.org/) a couple of times before and I never went very far. It felt overly complicated and other code editors would do the job just fine. None of the reasons to use it &mdash; it has a lot of plugins, you'll be very fast when you get used to it &mdash; seemed good enough. Other editors also have tons of plugins and you can be fast when you get used to them. But then, the real reason to use it came up.<!--more-->

In the past months, I have worked in three different computers: a Windows laptop, an OSX laptop, an OSX desktop. All of them have different keyboards &mdash; and the difference of commands in Windows and OSX machines are big. The way to solve the problem was to use something that would behave the same no matter the environment. And the solution was Vim.

## How to get started

There are a few good tutorials about how to use the movement keys (going up and down, moving to beginning or end of a word, up and down a file etc). One of the coolest one is a [game](http://vim-adventures.com/) in which we move around collecting keys to improve the movement.

After getting used to it, the idea is to have a very simple `.vimrc` (or `.vimfiles` in Windows). Of course it is possible to use some popular ones available online. They are full of features and made by people who are using Vim for years. But I realized it is better to start basic and just add features when they become necessary. This way, I know what things are doing and what I have available.

## The very basics of `.vimrc`

`.vimrc` holds the settings of Vim. Anything that you can do with `:command` while working, you can put on `.vimrc` so that you don't have to type it every time. For instance: `:set tabstop=2` corresponds to `set tabstop=2` on the file. This just sets how many columns a `tab` counts for.

`map` maps a shortcut to a command. `map <F1> :echo 'Current time is ' . strftime('%c')<CR>` will print the current time to the console every time we hit `F1` (the `<CR>` corresponds to *enter* or *return*; more key meanings can be found with `:help key-notation`). The `noremap` can be used instead of `map`. It does the same as `map`, but it won't change the mapping if a key has been mapped before.

`<Leader>` is mapped to a key (`\` by default). So `noremap <Leader> h :help<Enter>` will run `:help` when we enter `\h`.

## Installing plugins

There are a few plugin management tools and I chose to use [Vundle](https://github.com/VundleVim/Vundle.vim). Their [tutorial](https://github.com/VundleVim/Vundle.vim#quick-start) on how to install and use it is simple and easy to understand.

After the plugin is added to the `.vimrc`, we run `:PluginInstall` and done. There are plugins for pretty much everything you can think about. It is possible to have Vim set just like a GUI editor like Atom. A popular list of plugins can be found [here](http://vimawesome.com/).

## The same in all platforms

Adding plugins and customizations to Vim can lead to the original problem: having to use different commands in different computers.

One of the things I did was to remap `Control` key to the `caps lock`, *because it is easier to type*. It is much easier to hit `Ctrl-w` with that setting. However, I cannot remap the key in all the computers I have been using and it is not fun typing `W` when I actually wanted to change panes.

In the case of pairing, it is the same. Even if the other developer uses Vim, you cannot expect that they will also be used to hitting `caps lock` instead of `Ctrl` key (although this remapping is very common).

Even with those considerations, I still think that the remapping actually makes it easier to type, so I kept it.

Talking about pairing, I had pairing sessions with crafters here are [8th Light](http://8thlight.com) and they all used Vim. Although they offered to use another code editor, it was nice to be able to use Vim and learn a bit more about it. 
