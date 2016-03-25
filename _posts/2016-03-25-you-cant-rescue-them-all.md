---
layout: post
title: "You Can't Rescue Them All"
tags: [ruby, exceptions]
image: you-cant-rescue-them-all.png
---

Imagine you just woke up, took a shower and you immediately go to your coffee
machine to make that strong, large, morning, double-shot, extra-spice-and-everything-nice
cup of coffee. Sure, you go to the machine, press some buttons and the next thing
you know, you are waiting for the coffee to start pouring into your cup. And then,
something's not right, and something starts to smell bad. A morning nightmare, right?

## Nothing Works All the Time

You know, most of the things in our daily lives do not work at 100%. The nice
thing is that we never expect them to do that. Yeah, sure, if something breaks,
like that coffee machine, it might give us a slight frustration (especially in
the morning), but we won't lose our shit to it. We usually either try to fix it,
of course, by turning it off and on again. Or we just throw it away and buy a
new thing. And as a side-win, it's what drives our economy forward.

But what's really frustrating for us developers is that software does an 
exceptionally good job of misbehaving. Now this is where things can get hairy. 
Our software is not always prepared for errors. In fact we usually think about 
our code as this bulletproof titanium brick that does not fail under any 
circumstances. But more then often, our bulletproof software can get nuked. 
Houston, we have a problem.

## Handling Exceptions

What I really dislike about the whole culture in programming about exceptions,
is that it's being taught as this really unimportant thing. We study them briefly,
usually one chapter in a book and we think we are done with it. Yeah, until we
get hit by a "broken coffee machine" problem and then we try to deal with them.

In Ruby, we all know how to deal with exceptions. Sure, the syntax is nice:

{% highlight ruby %}
begin
  # Some HTTP requests here...
rescue Timeout::Error => e
  Logger.debug "Timeout error:"
  Logger.debug e.message
end
{% endhighlight %}

We wrap the code that we suspect will fail in a `begin` & `rescue` blocks and we 
can properly handle the exceptions. Sure, this works well, but I would like you
to focus on something else for this article - the exception class.

## What should we rescue?

I am sure you have seen some piece of code where someone knew that an exception
could be thrown, and that person thinks they've done a good job by rescuing 
`Exception`. 

{% highlight ruby %}
begin
  # Read a file
  # Parse the file
  # Send the parsed data to API
rescue Exception => e
  Logger.debug "Exceptions raised:"
  Logger.debug e.message
end
{% endhighlight %}

Right, all good? Sure, we could have done more with the `rescue` block, but at
least we log the exceptions somewhere so we can more easily view them in the 
future. There's a problem with this approach - you are essentially rescuing 
a metric ton of Ruby exceptions. And if this is a piece of code in a Rails 
application - the list is even longer. You are looking at couple of hundred 
exception types!

And if you think that it's fine, I have a question for you. Do you take the same
measures/actions when your car:

1. Has a blown up tire?
2. The engine does not want to start?
3. Door is frozen, because it's freezing cold outside?

Of course you don't. Fixing a blown tire by looking under the hood would be 
pretty silly, right? Well, it's the same as software - whatever goes wrong we
need to act accordingly, without generalisations.

For example, if a file will not open, there can be multiple reasons:

1. Permissions
2. Corrupt file
3. The stream is prematurely closed, or was never opened
4. File does not exist

Having all of these things in mind, we would need to write something like this:

{% highlight ruby %}
require 'json'

begin
  contents = File.read("users.json")
  JSON.parse(contents)
rescue IOError => e
  $stdout.puts e.message # 
rescue Errno::EACCES => e
  $stdout.puts e.message
  $stdout.puts "Will try to fix permissions and retry..."
  system('chmod +r users.json')
  retry
rescue Errno::ENOENT => e
  $stdout.puts e.message # File does not exist
  exit(1)
end
{% endhighlight %}

Sure, this might seem a bit overcomplicated for most of the cases that you will
see in your day job, but I think that it paints the picture well. Our software
should know how to take care of different cases of misbehaviour.

## Outro

As a rule of thumb, you should keep your rescues short and sweet. And very
specific. You should think about in what ways your program can misbehave and how
you can recover from that error. Sure, it definitely is not that easy, but if
your throw couple of random values at your code in the tests, you can easily
spot some of the misbehaviours.

In the end, I'll leave you with this quote, from Betrand Meyer, from his book 
*Object Oriented Software Construction*:

> In practice, the rescue clause should be a short sequence of simple 
> instructions designed to bring the object back to a stable state and to either 
> retry the operation or terminate with failure.

And if you want to learn more about exception handling in Ruby, I recommend 
[Exceptional Ruby](http://exceptionalruby.com/), by Avdi Grimm.
