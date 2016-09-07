---
layout: post
title:  "Running shell commands with Clojure"
date:   2016-09-07 13:50:32 -0500
category: apprenticeship
tags: [clojure]
---

One of the requirements of the Tic Tac Toe was that the user should be able to quit a game. Simple, right? The detail was that it should not only quit the game, but also clear the terminal screen. <!--more-->

That story, as simple as it was, helped me learn more about how to run shell commands using Clojure. There are two situations in which I needed to access the shell: getting the width of the console, used to centralize the output, and to clear the screen after quitting the game.

The first thing to do is to require `clojure.java.shell`, that starts a sub-process providing an input and collecting an output. One of the functions provided by the library is `sh`, that passes a string to `Runtime.exec()` (this will interact with the environment our application is running in and execute the command &mdash; the string passed to it).

```clojure
(ns blog.example
  (:require [clojure.java.shell :as shell]))

(->> (shell/sh "/bin/sh" "-c" "clear <  /dev/null") :out print))

; the line above uses thread last macro and could be expanded to this:
;(print "" (:out (shell/sh "/bin/sh" "-c" "clear <  /dev/null")))
```

The above command says to use shell (`"/bin/sh"`), pass a `"clear"` command to it (the `-c` flash says that the command are read from a string) and pipe any possible output to a `/dev/null` in order to silent it. This will run the `clear` command and will return a map:

```clojure
; `:exit` is the exit code; `:out` is the `stdout` and `:err` is the `stderr`

{:exit 0, :out "
", :err ""}
```

We then get the `:out` from that. It will give us some extra new lines, and calling `print` on it, with an empty space, avoids this to happen.

This of course will not work in Windows command line.

Here is the final function that clears the screen and quits the game:

```clojure
(defn clear-and-quit
  []
    (when-not (helpers/is-windows-os?)
      (->> (shell/sh "/bin/sh" "-c" "clear <  /dev/null") :out (print ""))
      (flush))
    (System/exit 0))
```  

The other situation in which I need to run a shell command was to get the console width. In this case, it took me a lot of research to find this solution (I didn't come up with it).

```clojure
(defn get-console-width
  []
  (->> (shell/sh "/bin/sh" "-c" "stty -a < /dev/tty")
        :out (re-find #"(\d+) columns") second))
```

Running `stty -a` returns a lot of information, including the line `25 rows; 80 columns;`. All this information will be on the `:out` field of the `sh` returned map. From there, we find the number of columns (`["80 columns" "80"]`) and get the second element from that array: `"80"`.

Once again, it won't work on Windows. The final function that gets the half size of a console is this:

```clojure
(defn get-half-screen-width
  []
  (if (helpers/is-windows-os?)
    40
    (quot (Integer/parseInt (get-console-width)) 2)))
```
