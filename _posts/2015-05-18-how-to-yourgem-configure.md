---
layout: post
title: "How to: YourGem.configure"
tags: [ruby, gems, patterns]
featured_image: /images/cover1.jpg
---

Really cool gems, like Carrierwave for example, have this neat feature of configuring
the gem in runtime. It allows you to easily configure how the gem will behave in your app.
For example, you can add various authentication keys, how errors should be handled and what not.
If you want to add this cool functionality in your gems, read on to find out more.


Personally, I love to implement (and use) this way of configuring libraries in runtime.
Although people have used and blogged about this "pattern" so many times,
I am writing another how-to because I really like this type of configuring external libs/gems.
So, if you've ever wondered how to implement it, here is another how-to.

## Randomizer

Lets build a gem that generates random numbers. We'll call it Randomizer. Basically,
every gem is a module, that namespaces bunch of classes and methods. So, say we
want our gem to have one method, called generate! that will return our random number.
Under the hood, we'll just use the built in rand method.

{% gist feb5e40196d46fac2171 %}

This is all good, the rand method will generate a really long Float. But, lets say
that we want the generate! method to return a number up to a maximum number? We can
change the method to accept a number as an argument and pass that number as an
argument to the rand method.

{% gist e876cc523316b3202488 %}

Again, really simple. But, what if we want to set a range for the random number
that will be generated? For example, for whatever reason an user might want the
method to return random numbers between 50 and 60. How can we do this?

Well, one approach is to make the generate! method accept two arguments, which is fine.
But, instead of passing the arguments every time we make the call, we can add a
configuration block so the gem always returns random numbers in a range. This is where
we need to introduce the configure block.

{% gist 27ccce27518020261d99 %}

How does this work? The configure class method stores a Configuration object
inside the Randomizer module. Anything set from the configure block is an
attr_accessor on the Configuration class. Now, lets add the generate! method back in
and see how we can use the configure block.

{% gist 4549dd0bee64bfb0507f %}

Now, the generate! method takes the from and to attributes from the
configuration object and creates an array of it. Than, we use the Array#sample method
which picks a random item from an array object.
So, say you have a Rails app where you want to use the Randomizer gem. Configuring
the gem can be done by adding a randomizer.rb file in the config/initializers folder.
Then, we need to use the configure block in the file.

{% gist f6a60eb05b992bb130fe %}

Now, when we call Randomizer.generate! the method will return a random number between 100 and 200.
Hope my explanation was good and you understood how you can use this "pattern".

What do you think, how we can adjust the gem so the method returns an array of two random numbers?
And how we can adjust the gem so the method will return a custom number of random numbers in an array?

Feel free to share your solution in the comments. Thanks for reading!
