---
layout: post
title: "Express yourself"
tags: [ruby, expressiveness]
---

In this blog post I am not going to talk about N.W.A's song called "Express Yourself". 
Nor Madonna's. Nope. I'll talk about Ruby and how wonderfully expressive it is. But,
we all know that, don't we? We have heard that tens or hundreds of times. But, I am 
not really sure we all know what that means. 

## Back to the future

About two years ago, when I was pair-programming with a more exprienced colleague of mine, 
I wrote the following code:

{% highlight ruby %}

if object.class == 'SomeClass'
  # do something...
end

{% endhighlight %}

As I wrote the code, he said to me:

<blockquote>
  Listen, that's okay. But, Ruby's expressive, so..
</blockquote>

I was very inexperienced, but I just interrupted him by saying:

<blockquote>
  Yeah, I know, it's awesome!
</blockquote>

He replied back with:

<blockquote>
  Well, why aren't you using it's expressiveness than?
</blockquote>

I got really confused. I was starring blank at the screen for couple of seconds,
thinking that I am doing something very wrong.  With a mild smile on his face and he said:

<blockquote>
  Yo, it's fine. You're not doing anything wrong. Everybody knows that Ruby's very
  expressive but not everyone uses it. Let me show you what I mean by that.
</blockquote>

As I passed the keyboard to him, he changed the code to:

{% highlight ruby %}
if object.is_a?(SomeClass)
  # do something...
end
{% endhighlight %}

I said:

<blockquote>
  Yeah, you're right... Thanks.
</blockquote>

And we just went on to finish the pairing session. But after thinking a bit about it
I understood that the expressiveness of the language is really one of it's greatest
powers. Ruby's made for programmer's happiness and realizing that instead of 
writing very verbose code you can use a built-in method makes me very happy.

## Don't reinvent the wheel

We often reinvent the wheel. Sometimes it's due to ignorance, other times it's due
to our will to learn how something's implemented. And in whichever group you fall in,
it's okay. If you don't know something - you'll learn it. If you fall into the second
group, well, very good, keep up the good work. Just remember to fall back to the 
built-in methods, because they are faster than your imeplementation 99% of the time.

I often write if statements that have this format:

{% highlight ruby %}
if post.state == :published || post.state == :archived
  # do something..
end
{% endhighlight %}

And it works. But, I always try to make my code more readable. Think about this:

{% highlight ruby %}
if [:published, :archived].include?(post.state)
  # do something...
end
{% endhighlight %}

Instead of gluing couple of predicates with the `||` operator, you can use the
`Array#include?` method. Yeah, I guess a lot of you have seen this technique,
but it's basically showing what you can do with just a bit of knowledge of Ruby's
built-in methods.

Another example is this. Let's find all of the key/value pairs in a Hash whose
value is an even number:

{% highlight ruby %}
hash = { a: 1, b: 2, c: 4, d: 5 }

even_hash = {}

hash.each do |key, value| 
  if value % 2 == 0
    even_hash[key] = value
  end
end

puts even_hash

# Output:
# {:b=>2, :c=>4}
{% endhighlight %}

This works fine. Right? It returned `{:b=>2, :c=>4}`. Sure it did, but look 
at the code. We are rebuilding a **brand new hash** just to get the pairs with
the even values. 

Let's try to refactor this a bit. Take a look at [Enumerable#inject](http://ruby-doc.org/core-2.2.3/Enumerable.html#method-i-inject) and let's try using it:

{% highlight ruby %}
hash = { a: 1, b: 2, c: 4, d: 5 }

hash.inject({}) do |store, hash|
  if hash.last % 2 == 0
    store[hash.first] = hash.last 
  end
end

{% endhighlight %}

This will work too. Although it looks better, we are building a new hash once again. 
Also, it's not really expressive. I mean, what's `.first` and `.last`? 
We can do 5 more refactoring steps to get to a really readable state of the code, 
but let's keep this short.

What you I am aiming for is:

{% highlight ruby %}
hash = { a: 1, b: 2, c: 4, d: 5 }

hash.select {|key, value| value.even? }

# Output:
# => {:b=>2, :c=>4}
{% endhighlight %}

See, Ruby's core team implemented [Enumerable#select](http://ruby-doc.org/core-2.2.3/Enumerable.html#method-i-select).
And if you didn't know it before, it's fine, just head over to the documentation 
and you'll learn it in two minutes. It's awesome. 

But also, look at the second part of the code: `value.even?`. Instead of checking
if the number is divisible by two, Ruby has the [Fixnum#even?](http://ruby-doc.org/core-2.2.0/Fixnum.html#method-i-even-3F) method. It's really simple. It returns `true` if the number it's called on 
is an even number. You would agree, very self-explanatory, right?

## Express yourself

We can go on and look at more examples of Ruby's built-in methods. But the whole 
point of this blog post is not to explain Ruby's methods. It's about understanding
the power that lies below the surface. We can always reinvent the wheel, especially
with easy stuff like the examples above. But below the surface, there's a huge spectrum
of methods that will make your code more readable and your life easier. And your 
colleague (or a future you), that will most likely read the code in couple of months/years,
will be thankful to you. 

So go ahead, open some code in your editor, look at simple if statements and loops 
and try to find a suitable replacement for the logic in them in Ruby's documentation.

Learning Ruby's expressiveness is easy, it just takes some practice.

