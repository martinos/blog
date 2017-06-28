+++
date = "2017-06-08T21:47:37-04:00"
title = "Function Composition in Ruby"
draft = true
+++

Two years ago I started to learn Elm and I quickly felt in love with this very nice functional language. One of the most usefull think that I discoverd was the `>>` and `<<` operator. They are called the function composition operators.

These operator are used to combine functions together. What does this mean? 

We are going to look at a small example.

Let say that we want to convert farenheights to degrees and convert that number to a string with the unit appended to it.

```ruby
fahrenheit = 70 
celcius =  (fahrenheit - 32) * 5.0 / 9
celcius_str = "#{celcius} celcius"
# => "21 celcius"
```
In order to make code more reusable you might extract 2 functions:
```ruby
def to_celcius(fahrenheit)
  (fahrenheit - 32) * 5.0 / 9
end

def celcius_str(deg)
  "#{deg} celcius"
end
```
Then you would:
```
celcius = to_celcius(fahrenheit)
celcius_str(celcius)
```

You can rewrite it this way

```ruby
celcius_str(to_celcius(fahrenheit))
```
We can create a lambda to reuse it.

```ruby
to_celcius_str = -> f { celcius_str(to_celcius(f)) }
```

This pattern of creating a function from two other functions is called function composition.  Function composition removes the need of using temporary variables, It makes code easier to write (no need to find the proper temp variables names) and easier to read (less noise in the code).

Since function composition, is everywhere why don't we create a special operator. But first let's look at the celcius display using functions:

If we rewrite all this using lamdas:

```ruby
to_celcius = -> fahrenheit { (fahrenheit - 32) * 5.0 / 9 }
celcius_str = -> deg { "#{deg} celcius" }
celcius_str.(to_celcius.(fahrenheit))
```

Let's create a special function composition operator. In order to do this we will need the superators19 gem.

```ruby
require 'superators19'

class Proc
  superator ">>~" do |fn|
  -> a { fn.(self.(a)) }
  end
end
```

I used the `>>~` operator because `>>` was not available. This operator take a function on the left side and function on the right side and returns a function that is the composition of these two functions.

Using this operator we can now rewrite the function that we are interested:

```ruby
fahren_to_celcius_str = to_celcius >>~ celcius_str
```
This new variable is a function that takes the temperature in fahrenheit and output a string that displays the temperature in celcius with its unit. It's equivalent to this:

```ruby
fahren_to_celcius_str = -> f { celcius_str.(to_celcius.(a)) }

```

Since the `>>~` operator returns a function with one params, you can chain the results with other functions.

Lets define a new function that output a string in red.

```ruby
red = -> str { "\033[31m#{str}\033[0m" }
```

Then you can redefine conversion function like this

```ruby
to_celcius >>~ celcius_str >>~ red
```

Then you will get a function that takes fahrenheit degrees and returns a red string containing the corresponding celcius value with the celcius units.

One of the really powerful property of function composition is associativity.

```ruby
to_celcius >>~ celcius_str >>~ red
```
is the same as
```ruby
(to_celcius >>~ celcius_str) >>~ red
```
and 
```ruby
to_celcius >>~ (celcius_str >>~ red)
```

So you can easily reuse parts of these expression by just putting a series of composition in a variable.

```
colored_celcius = celcius_str >>~ red
```

It's way less code manipulation than if you do the same thing in the imperative way.

```ruby
celcius = to_celcius(fahrenheit)
str = celcius_str(celcius)
red_str = red(str)
```

```ruby
def colored_celcius(celcius)
  str = celcius_str(celcius)
  red(str)
end

celcius = to_celcius(fahrenheit)
red_str = colored_celcius(celcius) 
```
This is a lot of manipulation compared of using the `>>~` operator. Note that temporary variables are used everywhere in imperative and oo programs. Since this pattern is everywhere, FP programs are containing way less noise than an FP program. They are shorter and clearer.
