---
title: The magic of enumerators
date: 2014-11-12 19:04 UTC
tags: enumerators, ruby
---
<br>

### What are enumerators?
Whenever I try and explain enumerators to people, they tend to have no idea what I'm even referring to.
Enumerators are objects that many enumerable methods return if they have not been given a block.

### How do I use an enumerator?
The first step is to grab an enumerator to play with.
I'm going to start with the most simple one for now:

```ruby
enumerator = [1,2,3].each
puts enumerator.inspect

# >> #<Enumerator: [1, 2, 3]:each>

```

Now that we have an enumerator, let's check out what methods it has.
Looking at the
[Ruby docs](http://ruby-doc.org/core-2.1.4/Enumerator.html),
we have:


```ruby
# each
# each_with_index
# each_with_object
# feed
# inspect
# next
# next_values
# peek
# peek_values
# rewind
# size
# with_index
# with_object
```

While many of these are very useful, I'm only going to go over a few.
My personal favorite of these methods is the:

### .with_index

I find this method to be very useful in some cases,
and I usually use enumerators for this functionality only.
In most ruby code, you don't need the index you're currently at because
enumerables take care of those problems very well.
However, you start needing every once in a while, especially when working with algorithms
and want your code to still look like decent, thought-out ruby code.

#### What does it do?

.with_index passes the index as an argument to the block you give it after
the typical arguments of the enumerator you're using.
For example:

```ruby
enumerator = ["a","b","c"].each

enumerator.with_index do |element, index|
  puts "#{element} is at #{index}"
end

# >> a is at 0
# >> b is at 1
# >> c is at 2
```

Ok, you understood that much but where is this actually useful?
Any algorithms that is dependent on the position of the element,
such as a checksum for credit card numbers,
can be written much cleaner using the enumerator .with_index method.

#### Examples

One part of a check sum's calculation is doubling every other digit.
As Rubyists, we immediately go to enumerables to solve all of our problems.
A likely first answer is:

```ruby
some_array = []
[4, 5, 9].each_with_index do |digit, index|
  if index.even?
    some_array << digit * 2
  else
    some_array << digit
  end
end

puts some_array.inspect

# >> [8, 5, 18]
```

Sure we got our answer, but who likes to shovel things into another
array when you're already using an enumerable method?
Here's how you can do this with the .with_index enumerator method:


```ruby
some_array = [4, 5, 9].map.with_index do |digit, index|
  index.even? ? digit * 2 : digit
end

puts some_array.inspect

# >> [8, 5, 18]
```

Doesn't that just look a little bit nicer?
It's a bit more concise how Ruby should look,
and we're not shoveling things into an array inside of our enumerable method.
Even without the ternary, the .with_index makes this looks quite a bit better:

```ruby
some_array = [4, 5, 9].map.with_index do |digit, index|
  if index.even?
    digit * 2
  else
    digit
  end
end
```

I think that's enough time for that method,
let's look at a few more.

### .next and .peek and .take

These two methods are useful if you want to take one step at a time
while going through your enumerator.
Enumerable methods go through everything in the list,
but this is one way to see what's going on between all that.

#### What does it do?

.next will give you the next value the enumerator would "yield" to
(call the block) and it also iterates once.
If there's nothing else to iterate over, it raises a StopIteration.

```ruby
enum = [1,2,3].each

puts enum.next
puts enum.next
puts enum.next
puts enum.next

# >> 1
# >> 2
# >> 3

# ~> StopIteration
# ~> iteration reached an end
```

Then we have .peek which does the same as .next but it doesn't iterate at all.
It still raises a StopIteration when reaching the end though.

```ruby
enum = [1,2,3].each

puts enum.peek
puts enum.peek
enum.next
puts enum.peek
enum.next
puts enum.peek
enum.next
puts enum.peek

# >> 1
# >> 1
# >> 2
# >> 3

# ~> StopIteration
# ~> iteration reached an end
```

Then .take(n) will do the same as .next n times:

```ruby
enum = [1,2,3].each

puts enum.take(2)

# >> 1
# >> 2
```

And if you do need to go back, you have:

### .rewind

There's not much explination needed for this one, so here's an example:

```ruby
enum = [1,2,3].each

puts enum.next
puts enum.next
enum.rewind
puts enum.next
puts enum.next

# >> 1
# >> 2
# >> 1
# >> 2
```

All rewind does is reset the iteration back to the beginning.
Pretty straightforward.

### Time to get fancy

Alright, we went over some of the methods that enumerators give us.
Now how can we make some enumerators to work with?

Many enumerable methods will return us one when called without a block.
Here's a short list of common enumerable methods that return enumerators:

```ruby
[].collect
[].each
[].each_slice(num)
[].each_with_index
[].each_with_object
[].find
[].flat_map
[].group_by
[].map
[].max_by
[].min_by
[].reject
[].select
[].sort_by
```

We can also use Enumerator.new and give it a block to iterate over.
Here's a relatively simple example:

```ruby
enum = Enumerator.new do |yielder|
  (4..10).each do |num|
    yielder << num
  end
end

puts enum.take(5)

# >> 4
# >> 5
# >> 6
# >> 7
# >> 8
```

This yielder parameter is where we add our items to iterate over for our enumerator.
A nice thing about this is that we infinitely add items to it,
then iterate only a set number of times.
For example, if we wanted all the even numbers:

```ruby
all_evens = Enumerator.new do |yielder|
  i = 1
  loop do
    yielder << i if i.even?
    i += 1
  end
end

puts all_evens.take(5)

# >> 2
# >> 4
# >> 6
# >> 8
# >> 10
```

We are looping infinitely here,
but we can still take the first n even numbers since the enumerator only generates
numbers when we called the .take method.

#### So what's so fancy?

We can write a few complex algorithms much simpler using these enumerators,
especially those that are dependent on previous values.
The famous example of the fibonacci sequence can be generated quite easily without recursion.
Here's one way to construct this:

```ruby
fib = Enumerator.new do |yielder|
  2.times { yielder << 1 }
  one_before = 1
  two_before = 1
  loop do
    add_value = one_before + two_before
    two_before = one_before
    one_before = add_value
    yielder << add_value
  end
end

puts fib.take(10)

# >> 1
# >> 1
# >> 2
# >> 3
# >> 5
# >> 8
# >> 13
# >> 21
# >> 34
# >> 55
```

I tried to keep the more complex algorithmic pieces out of the way,
so we can focus more on the enumerator piece.
The enumerator can now give us as many fibonacci values as we want,
without having to regenerate everything in a recursive way for every value.

What I mean by a recursive way is something like this:

```ruby
def fib(n)
  return 1 if n <= 2
  fib(n - 1) + fib(n - 2)
end
```

That's shorter code you say?
Valid point, but the enumerator never has to call itself again, and is thus much faster.
I benchmarked the enumerator version and the recursive version to find only the 50th fibonacci number.
Here are my results:

```ruby
# recursion:  1275.593612 seconds
# enumerator: 0.000019    seconds
```

In other words, enumerators are super cool and everyone should them.
