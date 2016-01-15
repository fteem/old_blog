---
layout: post
title: "Variable hoisting in Ruby"
tags: [ruby, variables, hoisting]
---

Have you ever heard of hoisting? Well, regardless if you have or you have not, 
Ruby has an interesting hositing mechanism built-in. Let's take a dive and see
how it creates variables and do some experiments with it.

## Hoisting

What is hoisting? Well, according to Google "hoist" means to raise something. Apparently,
with with ropes and pulleys. At least, back in the day.

![http://i.imgur.com/9fXszXs.jpg](http://i.imgur.com/9fXszXs.jpg)

Well, when it comes to variable hoisting, it's basically a mechanism by which the 
language, in our context - Ruby, declares and defines variables. Okay, that sounds 
cool. Well, there are couple of quirks in Ruby that you should be aware of. 

## Bit of weirdness

Take this code for example, and run it in console:

{% highlight ruby %}
# weird_1.rb
puts x
{% endhighlight %}

We will get an error, obviously. `x` is not defined therefore you get an error:

{% highlight ruby %}
NameError: undefined local variable or method `x' for main:Object
{% endhighlight %}

That is normal. Next, try this:

{% highlight ruby %}
# weird_2.rb
if true
  x = 1
end
puts x
{% endhighlight %}

This will output 1, which is expected. Now, let's try something else:

{% highlight ruby %}
# weird_3.rb
if false
  x = 1
end
puts x
{% endhighlight %}

What should this output? An error? Or nil? Or maybe 1, if somehow we live in a 
parallel universe? If we try to run this code, we will get a `nil`. You don't believe me?

Try adding this to the bottom of the script:

{% highlight ruby %}
# weird_4.rb
puts x.class
#=> NilClass
{% endhighlight %}

You see, if we try to call an undefined variable we will get a `NameError`. 
But, if we define the variable in a part of the code that **will not be run at all** 
we will get a nil. 

What on Earth is going on here, right? **Right?!**

## Hoisting

Well, it's not that complicated really. But, it's a quirk that most of us have not
ran across. The "magic" here is done by Ruby's parser.

Basically, when the parser runs over the if-clause (look in weird_3.rb example file)
it first declares the variable, regardless of whether it will actually be executed.
This means that when the parser sees `x=1`, it will actually declare the variable,
by assigning it to `nil` and let the interpreter than figure out if the `x = 1`
line will ever get executed.

Don't confuse the parser with the interpreter. The parser does not care whether `x` 
ever gets a value. The job of the parser is just to go over the code, find any local 
variables and allocate "space" for those variables. More specifically, set them to `nil`.

On the other hand, it's the interpreter that will follow the logical path of the program
and see if/when `x` will get a value and act on it.

Last but not least, if you know about hoisting in JavaScript, it's worth mentioning
that Ruby's hoisting is much different. In Ruby every variable is available only after the line where the variable has been assigned, regardless if that line will be executed or not.

<br/> <br/> <br/>
<small>
Update #1 - 21 Aug 2015: Article updated with Pablo Herrero's point on Ruby's v.s. JavaScript's hoisting.
</small>
