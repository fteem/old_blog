---
layout: post
title: "Rack: First Principles"
tags: [ruby, rack]
---

About three years ago, when I started working with Ruby and Rails, I noticed that
the term "Rack" always came up in my Google searches.
Overwhelmed with all of the stuff I needed to learn combined with the awesomeness of Rails, which
shields the new Rails devs from it's internals, I never really understood Rack or
writing Rack apps. Although I used to see people mentioning "middleware" or "Rack middleware"
I never really wrote (or tried to write) any middleware. But, you know what's really funny?

As a Rails developer you use Rack **every single day**, it's just that you're not aware of it.
So, for anyone that is wondering what Rack does and how you can leverage it's awesomeness, lets
take a look at Rack from first principles.

## What is Rack?

Wait, isn't it Rake, not Rack? Nope, [Rake](https://github.com/ruby/rake) is "Make for Ruby" and
it's basically a task runner. On the other hand Rack is something else.

Taken from [Rack's documentation](http://www.rubydoc.info/github/rack/rack/master/file/README.rdoc){:target='_blank'}:

<cite>
  Rack provides a minimal, modular, and adaptable interface for developing web applications in Ruby.
  By wrapping HTTP requests and responses in the simplest way possible, it unifies and distills the
  API for web servers, web frameworks, and software in between (the so-called middleware)
  into a single method call.
</cite>

What does this tell us? It wraps HTTP requests and responses. Okay, so you
can actually use the request (as an object) and write/create the response which the
Rack app will return. Cool? Next, has an API for servers, frameworks and middleware
into a single method call.

## A single method call?

Yes, a Rack application is **an object** that responds to a method called ```call```.

Taken from [Rack's specification](http://www.rubydoc.info/github/rack/rack/master/file/SPEC){:target='_blank'}:

<cite>
  A Rack application is a Ruby object (not a class) that responds to ```call```. It takes
  exactly one argument, the environment and returns an Array of exactly three values:
  The **status**, the **headers**, and the **body**.
</cite>

What's the environment? Well basically, it's the environment of the server, because Rack is
an interface (or an API) for web servers. More specifically, it's a Hash that has a lot
of headers (key-value pairs). Also, it contains some Rack specific variables. You
can explore the environment [here](http://www.rubydoc.info/github/rack/rack/master/file/SPEC#The_Environment){:target='_blank'}.

Why does it return exactly three values? That's what always the web clients are interested
in when receiving the response from the server:

* the status - did the request fail, or was it ok
* the headers - [HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers){:target='_blank'}. Rarely mentioned, but they are great!
* the body - the useful stuff that the server returns (usually a HTTP page or JSON payload)

Makes sense, right?

## Rack application

Let's see how we can create a simple Rack application.

First, we need to install Rack. Believe it or not (sarcasm sign!) Rack is a gem and we can
install it by running this command in our command line:

{% highlight bash %}
gem install rack
{% endhighlight %}

After you have Rack installed, you can use Rack in two ways. What's interesting is that a Proc
can be a Rack application. But why Proc? Well, if you remember, a Rack application
is an object that responds to the ```call``` method. ;-)

{% highlight ruby %}
# rack_app.rb
require 'rack'

rack_app = Proc.new do |env|
  [200, {'Content-Type' => 'application/json'}, ["{'response':'OK'}"]]
end

Rack::Handler::WEBrick.run rack_app
{% endhighlight %}

If we run this app in our command line:

{% highlight bash %}
ruby rack_app.rb
{% endhighlight %}

we can see that a webserver booted and it ran the Rack application. This is the output
of my webserver (the output of yours will vary):

{% highlight bash %}
[2015-06-16 21:26:38] INFO  WEBrick 1.3.1
[2015-06-16 21:26:38] INFO  ruby 2.0.0 (2014-09-19) [x86_64-darwin13.4.0]
[2015-06-16 21:26:38] INFO  WEBrick::HTTPServer#start: pid=12649 port=8080
{% endhighlight %}

As you can see, Rack started a WEBrick webserver that is running the Rack application we wrote.
This happens because WEBrick is built in into Rack, as a Handler, therefore the last line in
the Ruby file is:

{% highlight ruby %}
*** snip ***
Rack::Handler::WEBrick.run rack_app
{% endhighlight %}

Another way to run this applciation is using the ```rackup``` command. To use this command
we'll need to modify the application a bit:

{% highlight ruby %}
# rack_app.ru
app = Proc.new do |env|
  [200, {'Content-Type' => 'application/json'}, ["{'response':'OK'}"]]
end

run app
{% endhighlight %}

Much simpler, but what happened? Well, the Proc object still stands in, but the rest
of the stuff is gone. As you can see also the filename changed. Why?

First and foremost, when using the ```rackup``` command we need a "rackup" file, whose
extension is ***.ru***. Yes, just like Russia's TLD! Also, we use the ```run``` method
which basically tells ```rackup``` to call ```call``` on the object that ```run``` takes as
a parameter (or, the app).

Booting the server and running the app is trivial, just like before:

{% highlight bash %}
rackup rack_app.ru
{% endhighlight %}

The output, again, is very similar:

{% highlight bash %}
[2015-06-16 21:37:09] INFO  WEBrick 1.3.1
[2015-06-16 21:37:09] INFO  ruby 2.0.0 (2014-09-19) [x86_64-darwin13.4.0]
[2015-06-16 21:37:09] INFO  WEBrick::HTTPServer#start: pid=14930 port=9292
{% endhighlight %}

Oh, and did I mention you can cURL the app or view the app running in the browser?
WEBrick is a server, which means your Rack application is running on a webserver.
In fact, notice the last line of the log above - it says that the server is running on port 9292.
To see your Rack application in action, you can visit [http://localhost:9292](http://localhost:9292){:target="_blank" } in the browser.
Or, you can ```curl 127.0.0.1:9292```!

This is what I got when i *cURL*ed to localhost:9292:

{% highlight json %}
{'response':'OK'}
{% endhighlight %}

That's it. The server returned the response that the Rack application created.
There you go - this is a very tiny functioning Rack application.

## Outro

I hope this post helped you understand what Rack is and what it's used for. Next time
we will take a look at writing Rack middleware and integrating it into a Rails application.
Until than, feel free to share your thoughts on Rack with me and the ways you use it.
Also, I am looking for suggestions for the middleware example for the next post -
what kind of middleware would you like to me to cover?


Update 1 (2015-06-19): Thanks to [BlazeBoy](https://disqus.com/by/BlazeBoy/) for the Rack vs Rake suggestion. Thanks to [André](https://disqus.com/by/andresilveirah/) for noticing a typo.  


