---
title: Brute Forcing Logic Problems In Ruby
date: 2014-12-31 19:04 UTC
tags: ruby, logic, brute, force
---
<br>

## Intro

Today I'd like to explain a little about brute forcing a logic problem
(brute forcing meaning you try every possible solution).
This works better for logic problems that deal
with many "checks" that must be performed.
Here is an example problem:

> There are four houses in a row.
> They are made from the following materials:
> straw, wood, brick and glass.

> Richard's house is somewhere to the
> left of the wooden one and the
> third one along is brick.

> Roger owns a straw house and Hannah does
> not live at either end,
> but lives somewhere to the right
> of the glass house.

> Jess lives in the fourth house,
> whilst the first house is not made from straw.

> What order are the houses in and
> who lives in each house?

And here is an example of a
solution to the above problem

```ruby
HOUSES = [:straw, :wood, :brick, :glass]
PEOPLE = [:richard, :roger, :hannah, :jess]
POSSIBILITIES = HOUSES.permutation.to_a.product(PEOPLE.permutation.to_a)
                .map(&:transpose)

class Array
  [HOUSES, PEOPLE].flatten.each do |attr|
    define_method "#{attr}?" do
      include? attr
    end
  end
end

def correct_houses?(possibility)
  hannah_index = possibility.index(&:hannah?)
  possibility.index(&:richard?) < possibility.index(&:wood?) &&
  possibility[2].brick? &&
  possibility.find(&:roger?).straw? &&
  ![0, 3].include?(hannah_index) &&
  possibility.index(&:glass?) < hannah_index &&
  possibility.last.jess? &&
  !possibility.first.straw?
end

p POSSIBILITIES.find { |possibility| correct_houses? possibility }
```

Output:

```ruby
[[:glass, :richard], [:straw, :roger], [:brick, :hannah], [:wood, :jess]]
```

I'm not going to go over this, but I suggest you take
a short look at it to see the kinds of methods used here.
Methods like `.permutation` and `.product` are especially useful.

(and for those of you wondering,
the metaprogramming on the array class here is so I
have the `?` methods for a tiny bit of readability)

## The Approach

When looking at these kinds of problems,
the first thing I do is see how I want the data to look,
then generate the list of possibilities to match that.

In the example above,
I wanted the output to be a set of 4 houses,
and each set to contain the person and type.
"Set" in this case ending up as an array in code.
If you were answering some multiple choice questions,
you might want the output to look like:
`[:a, :b, :a, :c, :d]`

Now that we have some way to write the answer
in a way that ruby can understand,
we can generate every possible solution

Once we have the possible solutions,
we take all the "rules" that the problem
gives us and write them are booleans in our ruby code.
The goal we're going for here is where we can input the
`[:a, :b, :a, :c, :d]` and check whether it is correct or not.

Once we have those 2 piece's there's only 1 step remaining:
run through every possible solution and find the correct one.

## Example problem

Ok, that stuff sounds good and all, but how do we write it all?
I can't explain it any better than I did above
so let's just grab a problem and solve it.

> 1. The first question with B as the correct answer is:
>   * A. 1
>   * B. 4
>   * C. 3
>   * D. 2
> 2. The answer to Question 4 is:
>   * A. D
>   * B. A
>   * C. B
>   * D. C
> 3. The answer to Question 1 is:
>   * A. D
>   * B. C
>   * C. B
>   * D. A
> 4. The number of questions which have D as the correct answer is:
>   * A. 3
>   * B. 2
>   * C. 1
>   * D. 0
> 5. The number of questions which have B as the correct answer is:
>   * A. 0
>   * B. 2
>   * C. 3
>   * D. 1

Well this is the kind of problem that exactly
fits the kind that we can brute force.
Lots of possibilities with a bunch of intertwined logic,
that we can certainly check by running some boolean statements.

Anyway, our first step is to generate our possibilities.
We probably want the

`[:a, :b, :a, :c, :d]`

format so I'm going to stick with it.

We can generate an array of these by using the
`repeated_permutation` method.

To use this method, we need an array of possible answers,
(in this case `[:a,:b,:c,:d]`),
then just give the `repeated_permutation` method the
length that we want.
Since there are 5 questions, we want a length of 5.

```ruby
POSSIBILITIES = [:a,:b,:c,:d].repeated_permutation(5).to_a
```

output:

```ruby
[[:a, :a, :a, :a, :a], [:a, :a, :a, :a, :b], [:a, :a, :a, :a, :c]...]
```

Next, let's start writing checks for each of the questions.
The first question has to do with the first index of the answer b
we can get the index with `possibility.index(:b)`,
assuming possibility is the array of answers.
Now we just have to match that up the the answers the question gives us.
To get this done, I would use hash of answers like this:

```ruby
answers = {a: 1, b: 4, c: 3, d: 2}
```

This is pretty much stright copying from

> 1. The first question with B as the correct answer is:
>   * A. 1
>   * B. 4
>   * C. 3
>   * D. 2

The last part is to match the index and the answers up.
Here's what I ended up with:

```ruby
def q_1(possibility)
  answers = {a: 1, b: 4, c: 3, d: 2}
  answers[possibility[0]] - 1 == possibility.index(:b)
end
```

The `- 1` here is because the index method returns
us the array index which starts at 0.

### Skipping ahead: SPOILER ALERT

#### Before you read this next section, You may want to solve the `q_2` - `q_5` answers on your own, as I will not be going over them and the answer is here

Here are the answer checkers I have come up with
to check against all five questions:

```ruby
def q_1(possibility)
  answers = {a: 1, b: 4, c: 3, d: 2}
  answers[possibility[0]] - 1 == possibility.index(:b)
end

def q_2(possibility)
  answers = {a: :d, b: :a, c: :b, d: :c}
  answers[possibility[1]] == possibility[3]
end

def q_3(possibility)
  answers = {a: :d, b: :c, c: :b, d: :a}
  answers[possibility[2]] == possibility[0]
end

def q_4(possibility)
  answers = {a: 3, b: 2, c: 1, d: 0}
  answers[possibility[3]] == possibility.count(:d)
end

def q_5(possibility)
  answers = {a: 0, b: 2, c: 3, d: 1}
  answers[possibility[4]] == possibility.count(:b)
end
```

Now that we have all 5 checkers we only need to find
one possibility that will pass every check.
This is probably the simplest piece of code on this page,
and if you've made it this far, you can probably
figure it out on your own.
Either way, here's what I came up with:

```ruby
correct = POSSIBILITIES.find do |poss|
  (1..5).all? { |num| __send__ "q_#{num}", poss }
end
```

This code will run through all the possibilities,
"send" the methods `q_1`, `q_2`, etc,
and check if they were all true.

Finally, we can print out our answer:

```ruby
p correct
```

The answer we get now is `[:c, :d, :b, :c, :b]`

Congratulations! You've made it this far down this post.
I don't have much else to say other than good luck,
and here's the full code for the example for reference.

```ruby
POSSIBILITIES = [:a,:b,:c,:d].repeated_permutation(5).to_a

def q_1(possibility)
  answers = {a: 1, b: 4, c: 3, d: 2}
  answers[possibility[0]] - 1 == possibility.index(:b)
end

def q_2(possibility)
  answers = {a: :d, b: :a, c: :b, d: :c}
  answers[possibility[1]] == possibility[3]
end

def q_3(possibility)
  answers = {a: :d, b: :c, c: :b, d: :a}
  answers[possibility[2]] == possibility[0]
end

def q_4(possibility)
  answers = {a: 3, b: 2, c: 1, d: 0}
  answers[possibility[3]] == possibility.count(:d)
end

def q_5(possibility)
  answers = {a: 0, b: 2, c: 3, d: 1}
  answers[possibility[4]] == possibility.count(:b)
end

correct = POSSIBILITIES.find do |poss|
  (1..5).all? { |num| __send__ "q_#{num}", poss }
end

p correct
```
