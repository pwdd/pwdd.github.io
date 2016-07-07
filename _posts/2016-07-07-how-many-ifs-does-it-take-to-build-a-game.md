---
layout: post
title:  "How Many Ifs Does It Take to Build a Game?"
date:   2016-07-07 15:20:12   0500
category: apprenticeship
tags: [clojure]
---

While debugging the minimax implementation, I decided to create a record named `Game`, so the analysis of the board could be done accordingly to the type of a game: if it is human vs unbeatable computer, human vs human, and so on. In total, 6 types of games, and 10 conditionals, depending on the role of the two players.<!--more-->

The type of the game would be defined according to this:

```clojure
(defn game-type		
  [first-player second-player]		
  (cond		
    (= :human (role first-player) (role second-player))		
       :human-x-human		

    (= :hard-computer (role first-player) (role second-player))		
       :hard-x-hard		

    (= :easy-computer (role first-player) (role second-player))		
       :easy-x-easy		

    (and (= :human (role first-player))		
         (= :hard-computer (role second-player))) :human-x-hard		

    (and (= :hard-computer (role first-player))		
         (= :human (role second-player))) :human-x-hard		

    (and (= :human (role first-player))		
         (= :easy-computer (role second-player))) :human-x-easy		

    (and (= :easy-computer (role first-player))		
         (= :human (role second-player))) :human-x-easy		

    (and (= :easy-computer (role first-player))		
         (= :hard-computer (role second-player))) :easy-x-hard		

    (and (= :hard-computer (role first-player))		
         (= :easy-computer (role second-player))) :easy-x-hard		
    :else		
      :not-game-type))		
```

What if I had to add one more type of game? Would I have to add two more conditionals? And how to remember that a game between a human and an easy computer is `:human-x-easy` and not `:easy-x-human`? What if one more player was added? How many more conditional statements would be necessary? How many more tests should be written?

First decision was that the type of a game should be in alphabetical order. So, `:easy-x-human`, not `:human-x-easy`. Then, things got easier:

```clojure
(defn str-role
  [player]
  (if (= :human (role player))
    "human"
    (let [role (name (role player))
         limit (.indexOf role "-")]
      (subs role 0 limit))))

(defn str-game-type
  [first-name second-name]
  (keyword (clojure.string/join "-x-"(sort [first-name second-name]))))

(defn game-type
  [first-player second-player]
  (let [first-name (str-role first-player)
        second-name (str-role second-player)]
    (str-game-type first-name second-name)))
```

Now, if I have to add a player of type `parakeet`, it is easy to create a `Game` record with type `:human-x-parakeet`, no matter the order of the players.
