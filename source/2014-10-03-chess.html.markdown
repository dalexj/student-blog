---
title: Creating a chess game with gosu in ruby
date: 2014-10-03 19:04 UTC
tags: chess, ruby, gosu
---

First of all, what is this "gosu"? As I made my way through the beginnings of ruby, I wanted to go make something I can actually show someone that has no idea what coding is about. That means no using the command line and playing some mastermind, since they're not going to understand the work that goes into something like a command line app. So I asked around for some sort of way to display something outside of my console, and ended up hearing of this gem called gosu.

Simply, gosu's a (somewhat lightweight) 2D game engine with some very minimal functionality, but enough functionality for me to use it for a 2D board game. The first thing I noticed was that it's quite old and has not been updated for a ruby version past 1.9.3, so I had to grab that version before moving on. At some point, I may want to try and fork it and update to 2.1.2, but that's not what this article is about.

Because of how simplistic gosu is, it's very easy to pick up and learn in just a few minutes how to get the basics to show up on your screen. In my case, I merely want a few images of chess pieces to be placed on the screen and moved around at will, which only took me a few minutes to figure out how to import everything as gosu objects and draw them where I pleased. Once I had that all figured out, I moved onto the the actual chess playing.

A large section of my code has been directed toward merely algorithms to check if every piece is moving legally along the board. Looking at one piece by itself, this seems fairly simple to begin with. A rook on "A1" could move to any square on the board where one of the ranks/files matched. So all I needed to do to see if the rook could move was match its location by a regex of the target location. Example:

```ruby
def rook_can_move?(rook_location, desired_location)
  rook_location =~ Regexp.new("[#{desired_location}]")
end

rook_can_move?("A1", "A8")  # => 0
rook_can_move?("A1", "H1")  # => 1
rook_can_move?("A1", "B2")  # => nil
```

0 and 1 in this case will be "truthy" while nil is "falsey", which allows this simple regex to check if the rook is allowed to move to a certain spot. Now this type of "algorithm" needs to be written for every other piece in the game, but you also have to account for your king being in check, and not being able to jump pieces later. With either the rook or the knight here being the easiest to program a simple algorithm for, the rest ended up looking quite messy and I had to shove all those checker algorithms into their own files.

Eventually, after long hours of deleting code that didn't work and deleting more code that didn't work, I am now at a point where the game is completely playable except for 1 small feature of being able to choose which piece you want to promote to when your pawn reaches the end of the board.

It's been quite a ride going through trying to come up with the algorithms to a game I already know so well, and would recommend anyone to try and go through a similar process with games they know very well, because to a computer, your game is a whole lot more complex than you think it is.
