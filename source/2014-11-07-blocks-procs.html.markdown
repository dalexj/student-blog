---
title: Ruby blocks and procs
date: 2014-11-07 19:04 UTC
tags: ruby, blocks, procs
---
The enumerable methods are mainly what turns Ruby into such readable code. **I'm going to assume everyone reading this has at least a little experience using enumerable methods.**

Let's start out with something relatively simple:

```ruby
lengths = ["abc", "d", "ef"].map do |chars|
  chars.length
end
```

If we run this, we get:

```ruby
# => [3, 1, 2]
```

In this example, it is fairly easy to see what's going on.

How about this one?

```ruby
capitals = %w(a b c d).map(&:upcase)
```

If we run this, we get this:

```ruby
# => ["A", "B", "C", "D"]
```

### The first job of the &

Ok, most of you know the &:symbol shortcut already.
But who knows what it actually does?

First thing to note, the & turns a proc object into a block that can be passed to any method in Ruby.
Every method implicitly accepts a block, even if it's never used for anything.
Here's an example of how the proc is turned into a block rather than an object:

```ruby
to_s_proc = proc { |arg| arg.to_s }
[1, 2, 3, 4].map(&to_s_proc)
```

As expected:

```ruby
["1", "2", "3", "4"]
```

So the proc object we created was passed as a block to the .map method.

This proc object is just that => an object. So, it CAN be passed through one of the parameters of your function.

This is important to know because you muct pass it as a block to most functions.

```ruby
to_s_proc = proc { |arg| arg.to_s }
[1, 2, 3, 4].map(to_s_proc)

# => ArgumentError: wrong number of arguments (1 for 0)
```
the map method takes 0 arguments

The & turns something into a block. The block is just a set of code, not an object.

### The second job of the &

That's the first piece, the & also does one other thing behind the scenes.
It's calling .to_proc on whatever you're using it on.

In the case of a proc, this is pretty self explanatory:

```ruby
to_s_proc = proc { |arg| arg.to_s }
to_s_proc == to_s_proc.to_proc
```

If you're not using a proc, however, this feels like more Ruby magic.
In the case of a symbol, it has a very minimal .to_proc method.
Back to this example:

```ruby
capitals = %w(a b c d).map(&:upcase)
```
We know that a symbol is not a proc, yet it has this method on it:

```ruby
to_s_proc = :to_s.to_proc
to_s_proc.call(3)

# => "3"
```

So pretty much, anything that has a .to_proc method we can use as a block for our functions with the &:

```ruby
class StudentUpcaser

  def to_proc
    proc do |my_arg|
      puts "Doing my work..."
      my_arg.upcase
    end
  end

end

student = StudentUpcaser.new

%w(a b c).map(&student)
```

This code will return us:

```ruby
# => ["A", "B", "C"]
```

As well as printing out:

```ruby
# => Doing my work...
# => Doing my work...
# => Doing my work...
```

Now you can go out into Rubyland and create your own magical functions using procs with the &.
