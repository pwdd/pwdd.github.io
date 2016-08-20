---
layout: post
title:  "Adding colors to console output"
date:   2016-08-20 15:16:06 -0500
category: apprenticeship
tags: [git]
---

Just like the [minimax]({% post_url 2016-07-02-minimax-in-clojure %}), I thought that adding colors to the output of my Tic Tac Toe would be easy. However, as I had the output centralized based on the string length, it was not that easy. <!--more-->

The first thing was to create a namespace to deal with the colors:

```clojure
(ns ttt.colors)

(defn color-code
  [code-string]
  (let [escape (char 27)]
    (str escape code-string)))

(def colors-code-list
  (mapv color-code ["[31m" "[32m" "[33m" "[34m" "[35m" "[36m" "[37m"]))

(def colors-key-list [:red :green :yellow :blue :purple :cyan :default])

(def ansi-colors
  (zipmap colors-key-list colors-code-list))

(defn colorize
  [color string]
  (str (color ansi-colors) string (:default ansi-colors)))
```

It is very simple and everything would work OK, if it was not the fact that the ANSI color code would add to the string length, but as the escape sequence was not be displayed, the output centralization got messy.

## Counting a colorful string length

After a long while, I came up with some functions that would find the color sequence and count the string length without them:

```clojure
(def color-re #"\[\d?;?d*m")

(defn color-code-list
  [message-string]
    (re-seq color-re message-string))

(defn color-code-length
  [message-string]
  (if-let [color-list (color-code-list message-string)
           escape-chars (count color-list)]
    (+ escape-chars (count (string/join color-list)))
    0))

(defn message-length
  [message]
  (- (count message) (color-code-length message)))
```

## Removing colors from a string

The game worked fine, but all the tests that would expect strings in a certain way wouldn't pass. The tests were expecting things like `"Player 'x' moved to position 5"`, but it as getting `"Player '\[33mx\[37m' moved to position 5"`. The solution was to write a helper that would clean the strings specifically for the tests:

```clojure
(defn remove-color
  [string]
  (let [escape (char 27)]
    (loop [string string
           code-list (color-code-list color-re string)]
      (if (empty? code-list)
        string
        (recur (string/replace string (str escape (first code-list)) "")
               (rest code-list))))))
```
