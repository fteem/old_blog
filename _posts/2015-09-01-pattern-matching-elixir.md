---
layout: post
title: "Elixir through Ruby-tinted glasses: pattern matching"
tags: [ruby, elixir, pattern, matching]
---

Elixir has become the new hotness. And I think it's for a good reason. It's a 
functional language, it's fast and it really looks like Ruby. To some people 
the way the language "looks" shouldn't be much of a factor. But as a Ruby programmer 
I've become very attracted by a language's syntax. Ruby's syntax is awesome and I 
wouldn't want to learn a language with a hard-to-swallow syntax. 

So, if you still haven't read anything about Elixir, or if you are thinking of starting 
to learn it, keep reading on. Today I'll explore one of the very basic concepts in Elixir: matching.

Oh, and by the way, if you want to follow along head over to Elixir's 
[installation guide](http://elixir-lang.org/install.html), install it and come back to this post.

## Pattern Matching?

Yeah, pattern matching. What does it mean? If you immediately started thinking about pattern matching
with regex, stop! It's not that. 

Pattern matching is simillar to assignment in Ruby. Just simillar, not the same. Check this out:

{% highlight elixir %}
iex(1)> a = 1
1
{% endhighlight %}

Oh, btw, `iex` means **i**nterpreted **E**li**x**ir. It's like `irb`. 

So, that looks fine, right? `a` has the value of `1`. For the moment, let's say
that is "true". Check this out:

{% highlight elixir %}
iex(9)> a = 1
1
iex(10)> 1 = a
1
{% endhighlight %}

This didn't throw any errors, therefore it's a valid statement. But, what happened?
How is that a valid assignment? 

Well, it's **not** assignment, it's matching. And the equal sign is the **match operator**. It
basically matches (or asserts) that the left hand side matches the right hand side of the
operator. If it does, it bounds the value of the right hand side to the left hand side.

Check this out:

{% highlight elixir %}
iex(19)> a = 1
1
iex(20)> 2 = a
** (MatchError) no match of right hand side value: 1
{% endhighlight %}

The error's type is just that - a `MatchError`. It tells the programmer that there
was no match with the right hand side value (which is 1).

## Multiple matches

So, now that you understood the basics. Let's see something more interesting.

What does this do?

{% highlight elixir %}
iex(20)> [a, b] = [1, 2]
[1, 2]
{% endhighlight %}

Well, because the two **lists** have the same length, it will match the first item
on the left hand side with the first item in the list on the right hand side. 

This means that:

{% highlight elixir %}
iex(21)> a
1
iex(22)> b
2
{% endhighlight %}

Weird eh? Well, I am not an Erlang programmer, but as I understand this - matching comes 
from Erlang, and Elixir is built on top of it. Weird for us, Ruby programmers, but normal
for the others, I guess.

Since equals is matching, this won't work:

{% highlight elixir %}
iex(20)> [a, b] = [1, 2, 3]
** (MatchError) no match of right hand side value: [1, 2, 3]
{% endhighlight %}
It can't match the values in the left hand side list and the right hand side one because
the lenghts of the lists are different.

## Heads and tails

Nah, not gonna throw the coin. It's another cool feature of Elixir. How can we match
the first item in a list and the rest of the items in a list?

With Ruby, you could do something like:

{% highlight ruby %}
>> head, *rest = [1,2,3,4]
=> [1, 2, 3, 4]
>> head
=> 1
>> rest
=> [2, 3, 4]
{% endhighlight %}

On the other hand, in Elixir, this is done with the **pipe** (|) operator:

{% highlight elixir %}
iex(23)> [head|rest] = [1,2,3,4,5]
[1, 2, 3, 4, 5]
iex(24)> head
1
iex(25)> rest
[2, 3, 4, 5]
{% endhighlight %}

But also, take a note at the syntax. The pipe operator allow us to match the first
item and the rest of the list. But, we still match a list to a list, unlike Ruby.

## The underscore

The underscore keyword is an interesting one. It basically states that anything 
it matches is throw-away data. What do I mean? Check this out:

{% highlight elixir %}
iex(4)> [a, b, _] = [1, 2, 3]
[1, 2, 3]
iex(5)> a
1
iex(6)> b
2
iex(7)> _
** (CompileError) iex:7: unbound variable _
{% endhighlight %}

As you can see, _ is an unbound variable. It can match anything, but you cannot bind
a value to it. It's basically like redirecting output to `/dev/null` on a Unix
operating system. 


Well, that's about it for today. I hope this post added some fuel to your Elixir fire
burning deep inside. If you want to start learning Elixir, just grab Dave Thomas' 
[Programming Elixir](https://pragprog.com/book/elixir/programming-elixir).
And for the rest of you, Elixir developers, feel free to chime in the comments with any feedback.
