---
layout: post
title: "Testing Ruby's floats precision"
tags: [ruby, testing, tdd, floats, precision]
---

Float precision in Ruby is a well known quirk. But when testing floats, not many
of us bother to remember this and make their tests respectful to this quirk. In
this post we will see how the popular Ruby testing frameworks help us test Floats 
properly.

## Background story

Last week I published [a post about migrating a test suite from RSpec to Minitest](/migrate-rspec-to-minitest).
What was very interesting is that I got a mention on Twitter from [Ryan Davis](https://twitter.com/the_zenspider)
with an offer for a code review of the migration. Here's the convo:

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/fteem">@fteem</a> yay! lemme know if you want a code review or anything.</p>&mdash; Ryan Davis (@the_zenspider) <a href="https://twitter.com/the_zenspider/status/621084957612453888">July 14, 2015</a></blockquote>
<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/the_zenspider">@the_zenspider</a> thanks so much, you are awesome! :-)</p>&mdash; Ile (@fteem) <a href="https://twitter.com/fteem/status/621241362348933120">July 15, 2015</a></blockquote>

Ryan did the review for me, and one of his comments was:

<a href='https://github.com/fteem/forecastr/commit/39264e419dc932f4562d622a293d920634218af6#diff-3a10fd885974a1612768d9ccd36b7f13R10' target="_blank">
  ![Imgur](http://i.imgur.com/nC19wm9.jpg)
</a>

Lets see why...

## Ruby Float (im)precision

Float numbers cannot store decimal numbers properly. The reason is that Float 
is a binary number format. What do I mean? Well, Ruby always converts Floats from
decimal to binary and vice versa.

Think about this very simple example. Whats the result of 1 divided by 3? Yup, 0.33333333333... 
The result of this calculation is 0.333(3), with 3 repeating until infinity. 

This same rule, or quirk, applies to binary numbers. When a decimal number is converted 
to binary, the resulting binary number can be endless. Mathematically this is all fine. 
But in practice, my MacBook Air doesn't have endless memory. I am running just on 
4GBs of RAM, so Ruby must cut off the endless number at some point. Or it will fill 
up the whole memory of the computer and it will become useless. 
This mechanism of rounding numbers produces a rounding error and that's exactly 
what we have to deal with here.

So, the base rule about this is: **do not** represent currency (or, money) with Float.

## In practice

Take this for an example. Simple calculation. We want to add 0.1 to 0.05, which should 
return 0.15. Right? Lets give it a try:

{% highlight ruby %}
>> 0.1 + 0.05 == 0.15
=> false 
{% endhighlight %}

Okay, what? Let's see what's the result of the addition:
 
{% highlight ruby %}
>> 0.1 + 0.05
=> 0.15000000000000002
{% endhighlight %}

You can see that Ruby rounds off the number at the end. Lets print this number with 50 decimal points:

{% highlight ruby %}
>> sprintf("%0.50f", 0.10 + 0.05)
=> "0.15000000000000002220446049250313080847263336181641"
{% endhighlight %}

Whoa! You can see that the actual result of this addition is quite different from 0.15. 
Ruby here rounds off the numbers, because the difference is so "microscopic". 

If you are curious, here's how the numbers 0.10 and 0.05 actually look like with 
50 decimal points in Ruby:

{% highlight ruby %}
>> sprintf("%0.50f", 0.10)
=> "0.10000000000000000555111512312578270211815834045410"
>> sprintf("%0.50f", 0.05)
=> "0.05000000000000000277555756156289135105907917022705"
{% endhighlight %}

## Testing it

Okay, so now when the problem is obvious, how can we test it? 
The best way to test this is to use a **delta** number. You can think of this delta 
number as a number showing the margin of rounding error. 

For example, the delta for the 0.10 + 0.05 operation is approximately 0.0000000000000001.

### With Minitest

Minitest provides us the ```assert_in_delta``` and ```assert_in_epsilon``` methods. 

#### assert_in_delta 
It fails unless the expected and the actual values are within **delta** of each other.

{% highlight ruby %}
def test_precision
  assert_in_delta(0.15, 0.10 + 0.05, 0.0000000000000001)
end
{% endhighlight %}

This means that, while we will expect to get 0.15 as a result, the rounding-off error
can be as big as 0.0000000000000001.

If you see the source of this method, it's quite easy to understand:

{% highlight ruby %}
# File minitest/unit.rb, line 122
def assert_in_delta exp, act, delta = 0.001, msg = nil
  n = (exp - act).abs
  msg = message(msg) { "Expected #{exp} - #{act} (#{n}) to be < #{delta}" }
  assert delta >= n, msg
end
{% endhighlight %}

The delta must be larger or equal than the absolute value of the result of subtraction 
of the expected and the actual values.

#### assert_in_epsilon

This method behaves in a different matter than ```assert_in_delta```. For example,
if we use the delta as an epsilon, this test will fail:

{% highlight ruby %}
def test_precision
  assert_in_epsilon(0.15, 0.10 + 0.05, 0.0000000000000001)
end
{% endhighlight %}

{% highlight bash %}
# Running:

F

Finished in 0.001584s, 1262.6263 runs/s, 1262.6263 assertions/s.

  1) Failure:
SomeTest#test_with_epsilon [test.rb:5]:
Expected |0.15 - 0.15000000000000002| (2.7755575615628914e-17) to be <= 1.5e-17.
{% endhighlight %}

Why this happens is easier to see in the source of the ```assert_in_epsilon``` method:

{% highlight ruby %}
# File minitest/unit.rb, line 132
def assert_in_epsilon a, b, epsilon = 0.001, msg = nil
  assert_in_delta a, b, [a, b].min * epsilon, msg
end
{% endhighlight %}

So, ```assert_in_epsilon``` is a wrapper for ```assert_in_delta``` with a small but
important difference. The delta here is subject to "auto-scaling". This means that 
it will increase for the product of the smaller number from the expected value/actual 
value pair and the epsilon.

Also, I guess, it's called "epsilon" because the greek letter Epsilon is usually used to 
denote a small quantity (like a margin of error) or perhaps a number which will be 
turned into a zero within some limit.

### With RSpec

RSpec provides us the ```be_within``` matcher. The same rules apply here as the Minitest
```assert_in_delta``` method. The format is:

{% highlight ruby %}
expect(<actual>).to be_within(<delta>).of(<expected>)
{% endhighlight %}

Or, in our case:

{% highlight ruby %}
it "should match within a delta" do
  expect(0.10 + 0.05).to be_within(0.0000000000000001).of(0.15)
end
{% endhighlight %}

## Conclusion 

Since the people behind RSpec and Minitest are awesome, they have provided us these
methods where we can easily smooth out the edges of testing floats. What's very
important to understand here is that while testing this is pretty easy, it's 
**extremely** important to know what to use when. 

When it comes to money/currency, every sane developer out there will use [BigDecimal](http://ruby-doc.org/stdlib-2.1.1/libdoc/bigdecimal/rdoc/BigDecimal.html).
It provides arbitrary-precision floating point decimal arithmetic, which means that 
it will **always** get a correct result for any calculation involving floating point numbers.

As an outro, I'll leave you with this tweet:

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">&quot;If you&#39;re having float problems, I feel bad for you, son I got 99.0000001 problems but .9999 ain&#39;t 1&quot; - <a href="https://twitter.com/geofft">@geofft</a></p>&mdash; About Programming (@abt_programming) <a href="https://twitter.com/abt_programming/status/621334672228921344">July 15, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
