---
title: a = a is nil?????
date: 2014-10-26 19:04 UTC
tags: ruby
---

```ruby

a = b  

# ~> NameError: undefined local variable or method `b' for main:Object

```

Alright, this much makes sense, we have never set b, but take a look at this next one

```ruby

a = a      # => nil
a.inspect  # => "nil"

```
Wait a minute... What's going on here?

Above, we tried to call a variable that didn't exist and ended up with a NameError

One more silly thing about this, just to see exactly what's going on here.

```ruby

defined?(a)  # => nil
a = a        # => nil
defined?(a)  # => "local-variable"

```

This is just proving that we have not set the variable "a" beforehand, yet somehow it's being set to nil when we set it to itself

# Why does this happen? #

First of all, we need to realize that this is ruby, and ruby likes to do lots of magic behind our backs

Whenever we set a local variable inside ruby, it creates a placeholder for variable to go. This placeholder, as you may have guessed, is nil.

It's a little hard to explain exactly, so here's some code that shows what I'm talking about.

```ruby

begin
  a = b
  puts "a = b didn't throw an error"
rescue NameError
  puts "NameError, as expected"
end

# >> NameError, as expected

```

Ok this is what we showed earlier, right? But if you see here, the code still tried to set a to b, and continued on because we handled the error

Now remember what I said about ruby creating a placeholder for our variable? Let's use the above code to see that in action


```ruby

begin
  puts a.inspect
rescue NameError
  puts "a is not set yet"
end

begin
  puts "a being set to placeholder, but errors at b because b is not set"
  a = b
rescue NameError
end

begin
  puts a.inspect
rescue NameError
  puts "a is not set yet"
end

# >> a is not set yet
# >> a being set to placeholder, but errors at b because b is not set
# >> nil

```

Well that's a bunch of begins and resuces to read through, but the general idea is pretty straightforward.

When ruby sees:

```ruby
a =

```

It sets a right then and there to nil, our "placeholder".

When ruby sees:

```ruby
a = b

```

It first sets a to the "placeholder" nil, then errors out because b doesn't exist.
In our case, we handled the error, so we could keep going with our program, with a being set to nil

This brings us right back to where we started

```ruby
a = a # => nil

```

Now we can explain why this is happening.
At first, ruby sees that we want to create this variable "a" and sets it to nil momentarily.
Ruby then sees that we're setting it to a, and it knows a is nil, so a ends up as nil rather than undefined

# Wrapping up #

Well we uncovered one of the many ways ruby is trying to protect us, which hopefully will further your understanding of the language.
Ruby, while being extremely beginner friendly, ends up hiding a bunch of random things like this one as to not confuse us.
However, in small edge cases like this, we can run into a random unexplained pieces of the code that leaves us completely confused why it even works.
It's good to note that in these cases, there is usually an explanation which ends up not being too complicated for what's really happening behind the scenes.
