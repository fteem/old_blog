---
layout: post
title: 'Implementing "the lovely" Singleton Pattern'
tags: [ruby, design, patterns, singleton]
---

In every software, there are some things that have to be unique. For example, a Rails app has
only one logger. Also, applications must have configurations, like environment, various API keys and etc.
Take the configuration example - we need only one configuration for a runtime of an application. If
all of the configuration data is stored into a class, then the whole app will need to use
an object of that class. Hence, we use singletons - classes that can only have one object instantiated
from them.

Basically, singleton classes prevent instantiation of more than one object of that class.
Some people say that [singletons are bad](http://blogs.msdn.com/b/scottdensmore/archive/2004/05/25/140827.aspx),
some don't. But I am pretty sure there are fine use-cases that we need to be aware of.


There are couple of ways to implement this pattern in Ruby:

## Ruby Singleton Module

Ruby's Standard Library has a module that allows the creation of Singleton pattern. Using it
is really easy.

{% highlight ruby %}
require 'singleton'

class Configuration
  include Singleton

  attr_accessor :data

  def initialize
    @data = {}
  end

  def add key, value
    @data[key] = value
  end

  def version
    '0.0.1'
  end
end
{% endhighlight %}

Here, we create a Configuration class - a class that has a hash called 'data' that will
contain all the configration data. We added the ```add``` method that accepts a key and a value pair
as arguments. This method will add the key-value pair to the data hash.
{% highlight ruby %}
>> c = Configuration.new
NoMethodError: private method `new' called for Configuration:Class
  from (irb):27
  from /Users/ie/.rbenv/versions/2.0.0-p576/bin/irb:12:in `<main>'
{% endhighlight %}

As you can see, instantiating a new object of this class is prohibited. Accessing the singleton
is done using the ```instance``` class method.

{% highlight ruby %}
>> Configuration.instance
=> #<Configuration:0x007fc2799e1568 @data={}>

>> Configuration.instance.data
=> {}
{% endhighlight %}

Also, using the ```add``` method is done by calling it on the ```instance```:
{% highlight ruby %}
>> Configuration.instance.add "environment", "development"
=> "development"

>> Configuration.instance.data
=> {"environment"=>"development"}
{% endhighlight %}


## Instantiating a single object of a class

So, how can we reproduce the same class as a Singleton without mixing in the Singleton module?

{% highlight ruby %}
class Configuration
  attr_accessor :data

  def initialize
    @data = {}
  end

  def self.instance
    @@instance
  end

  def add key, value
    @data[key] = value
  end

  @@instance = Configuration.new

  private_class_method :new
end
{% endhighlight %}

So, where's the magic? The ```initialize``` and ```add``` methods look pretty usual.
We create a class variable ```@@instance``` that is an object of the Configuration class.
The key thing here is that we 'protect' the ```Configuration.new``` method by making it a private
class method. Then, ```Configuration.instance``` method will just return the ```@@instance```.
The use case is very much the same.

{% highlight ruby %}
>> Configuration.new
NoMethodError: private method `new' called for Configuration:Class
  from (irb):20
  from /Users/ie/.rbenv/versions/2.0.0-p576/bin/irb:12:in `<main>'

>> Configuration.instance
=> #<Configuration:0x007fd76419ebf8 @data={}>

>> Configuration.instance.add 'env', 'dev'
=> "dev"

>> Configuration.instance.data['env']
=> "dev"

>> Configuration.instance
=> #<Configuration:0x007fd76419ebf8 @data={"env"=>"dev"}>
{% endhighlight %}

## Constants and global variables
The key feature of a Singleton, beside the only-one-available-object is that the Singleton has to be
globally acessible. That's why constants and global variables can play well here.

{% highlight ruby %}
CONFIGURATION = Configuration.new
CONFIGURATION.add 'environment', 'development'
{% endhighlight %}

{% highlight ruby %}
$configuration = Configuration.new
$configuration.add 'environment', 'development'
{% endhighlight %}

One problem that global variables have is that it can get changed in runtime, without you noticing.

## Class
Singleton pattern can also be implemented by using class methods and variables.
By using class methods we have are sure that we will have a single instance of the class.

{% highlight ruby %}

class Configuration
  def self.add key, value
    @@data ||= {}
    @@data[key] = value
  end

  def self.data
    @@data ||= {}
  end
end

{% endhighlight %}

Using it is, again, simple:

{% highlight ruby %}
>> Configuration.data
=> {}

>> Configuration.add 'environment', 'production'
=> "production"

>> Configuration.data
=> {"environment"=>"production"}
{% endhighlight %}

## Module

Simillar to the class implementation, a module can be a Singleton. Also, by default,
instantiating an object from a module is impossible, which is a nice thing when it comes
to the Singleton pattern.

{% highlight ruby %}

module Configuration
  def self.add key, value
    @@data ||= {}
    @@data[key] = value
  end

  def self.data
    @@data ||= {}
  end
end

{% endhighlight %}

Using the module is the same as using the class:

{% highlight ruby %}
>> Configuration.data
=> {}

>> Configuration.add 'environment', 'production'
=> "production"

>> Configuration.data
=> {"environment"=>"production"}
{% endhighlight %}


## The good, the bad and the Singleton Pattern

There is [quite a debate](http://stackoverflow.com/questions/137975/what-is-so-bad-about-singletons)
[going on](https://practicingruby.com/articles/ruby-and-the-singleton-pattern-dont-get-along)
[the internet](http://jalf.dk/blog/2010/03/singletons-solving-problems-you-didnt-know-you-never-had-since-1995/)
[about the](https://jorudolph.wordpress.com/2009/11/22/singleton-considerations/) Singleton pattern. Some are in love with
it, others consider it an anti-pattern and despise it. What's very crucial that you must understand
is that every pattern has it's use cases. Yes, each and every one of them!

In my opinion, the problem with the Singleton pattern is that if you fall "in love" with it,
you might overuse it and then the coupling and the global state might hunt you for years.
On the other hand, the best examples where you should use Singletons that come to mind
are application Configuration and Loggers. So, like I said, it has it's use cases but you should
really think hard about it before using it.

When and where have you used Singletons? Is it that bad, or is it much better? What's your
opinion on using the Singleton pattern?
