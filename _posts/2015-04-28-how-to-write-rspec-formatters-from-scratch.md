---
layout: post
title: "How to write RSpec formatters from scratch"
tags: [angularjs, google, javascript]
---

Recently I did an experiment with RSpec's formatters. Turns out, the output that RSpec returns when you run your specs can be very customized for your own needs. Read on to learn how you can write custom RSpec formatters.

## Writing custom formatters

RSpec allows customization of the output by creating your own Formatter class. Yep, it's that easy. You just need to write one class and than require it into RSpec's configuration to use it. Lets explore couple of key concepts about the formatters and it's internals.

### The Protocol

First thing to be aware of is the protocol (the order) that RSpec uses when calling methods from the formatter. Keep in mind that every method receives one argument, which is a Notification object. This is sent by the RSpec reporter which notifies the formatter of the outcome of the example. On the beginning **Formatter#start **is called. This method takes a notification argument of the class [StartNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/StartNotification). On every example group, **Formatter#example_group_started** is called. This method takes a notification argument of the class [GroupNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/GroupNotification). When an example is ran, one of these methods is called, based on the result of the example:

*   If the example passes, **Formatter#example_passed **is called. The notification argument is of class [ExampleNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/ExampleNotification).
*   If the example fails, **Formatter#example_failed **is called. The notification argument is of class  [FailedExampleNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/FailedExampleNotification).
*   If the example is pending, **Formatter#example_pending **is called. The notification argument is of class [ExampleNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/ExampleNotification).

At the end of the spec suite, these methods are called in this order:

*   **Formatter#stop**. The notification passed as argument is of class [ExamplesNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/ExamplesNotification).
*   **Formatter#start_dump**. The notification passed as argument is of class [NullNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/NullNotification).
*   **Formatter#dump_pending**. The notification passed as argument is of class [ExamplesNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/ExamplesNotification).
*   **Formatter#dump_failures**. The notification passed as argument is of class [ExamplesNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/ExamplesNotification).
*   **Formatter#dump_summary**. The notification passed as argument is of class [SummaryNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/SummaryNotification).
*   **Formatter#seed**. The notification passed as argument is of class [SeedNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/SeedNotification).
*   **Formatter#close**. The notification passed as argument is of class [NullNotification](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Notifications/NullNotification).

I wont go into detail for every of the notification classes. You can dive in the details about each of them in the links that I have attached.

### Registering your formatter class to RSpec

RSpec provides this neat feature of registering a class as a formatter. This is done by creating the class and calling:

{% highlight ruby %}
RSpec::Core::Formatters.register <the class name>, <array of symbols representing the methods that this formatter will provide>
{% endhighlight %}

So, say we want a formatter that shows the progress of the suite, just like the built-in progress formatter, but it will group the failing and the pending specs in the summary of the suite. Oh well, a picture is worth a thousand words:

[![target](http://rubylogs.com/wp-content/uploads/2015/04/target.jpg)](http://rubylogs.com/wp-content/uploads/2015/04/target.jpg)

The picture shows what we'll be working on in the rest of this post. We'll call this formatter **GroupingFormatter**.

### Creating the the GroupingFormatter class

{% gist cd69c8a17396b055951d %}

As you can see, the GroupingFormatter is just a class. In it's initializer it takes the output as an argument and sets it as an instance variable. Also, on line 2, you can see the aforementioned **RSpec.register** call. We pass _self_ as the first argument, because we want to register this class as a formatter. The rest of the arguments are method names that RSpec will call when using this formatter. What this means is that when you define a method for **the protocol, **if you don't register it - it will not be called. Basically, RSpec won't know it exists at all. Next, the **dump_summary** method calls the duration method on the notification object, which returns a number representing the time of the specs' duration in seconds. So, how can we test if this is working? The command is:

{% highlight bash %}
rspec some_file.rb --require ./grouping_formatter.rb --format GroupingFormatter
{% endhighlight %}

And the output is:

{% highlight bash %}
Finished in 0.007552.
{% endhighlight %}

Now, this doesn't tell much. Let's use RSpec's built in helpers to format this number in a meaningful string.

{% gist 3a15204f4d6226a0aebd %}

In the **dump_summary** method we use the [RSpec::Core::Formatters::Helpers](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Formatters/Helpers) module which has some methods that can help us turn the duration number into a meaningful string. The output now looks like:

<pre>Finished in 0.00758 seconds.</pre>

Okay, great. Now, lets make this formatter mimic the reporting formatter that comes with RSpec. We need the formatter to show a dot for every passing example, F for every failing example and an asterisk for every pending example.

{% gist 0303873a6a7e66c0787b %}

So, the reporter (the algorithm that follows the protocol) will call **example_failed** when an example fails, **example_pending** when an example is pending and **example_passed **when an example passes. This is really self-explanatory - we add the case specific character to the output for every example. Take note that I added the method names to the **RSpec.register** call. If I didn't - they'd be ignored. The output will now look like:

<pre>.....FF*..

Finished in 0.0207 seconds.</pre>

Looking good, things are starting to take shape! Now for the more complicated part. How can we group the pending/failing specs? First, lets group the pending specs.

{% gist f89bbb3e28d00602ab50 %}

Lets look at the **dump_pending** method now. First, it adds "PENDING" to the output. Next, it loops through the _pending_examples_ array and creates an array of strings for each of the pending examples. Note that I added the new method to the RSpec.register call, it would be ignored otherwise. Each string in the array will look something like this:

<pre>Something is pending - ./something_spec.rb:90</pre>

At the end, we call _join _on the array of strings to build a single formatted string that we append to the output. Now when we run the specs with the formatter, the output will look like:

<pre>.....FF*..

PENDING:
  Something is pending - ./something_spec.rb:90

Finished in 0.01121 seconds.</pre>

Looking good. Now, for the trickiest part, grouping the failing specs and adding the error underneath every failing spec. 

{% gist 5c1cb442677c4bf9619d %}

In the new **dump_failures** method we loop through every failed example. Then, we extract the description and the location of the failed example and we build a string that we append to the output. After this change, the output will look like this:

<pre>.....FF*..

PENDING:
 Something - pending - ./something_spec.rb:90
FAILING
 Something - first that fails - ./something_spec.rb:82
 Something - second that fails - ./something_spec.rb:86</pre>

Next thing, how do we add the error messages underneath every failing spec? Lets expand the **dump_failures **method just a bit. 

{% gist c6bd22b07504e16de893 %}

The only addition is on line 34 - we extract the result of the execution of the example, then we get the message of the exception that RSpec raised when the example failed. Now, lets test it:

<pre>.....FF*..

PENDING:
 Boxer is pending - ./boxer_spec.rb:90
FAILING
 Boxer first that fails - ./boxer_spec.rb:82
expected: false
 got: true
(compared using ==)

 Boxer second that fails - ./boxer_spec.rb:86
expected: false
 got: true
(compared using ==)

Finished in 0.0203 seconds.</pre>

This is all good, but you can see that the text alignment is broken a bit. If you look at the picture at the beginning, you will notice that the exceptions should appear indented underneath the description of the failing example. Lets fix this. 

{% gist 3c4d9a5da0827ac63a32 %}

In the example above we took the extra step to format the error messages nicely. Basically, we split the exception message on a new-line character, we remove all the whitespace and we rejoin the pieces with a newline between them and add 10 spaces at the beginning of the message (for the indentation). Now, the output will look like this:

<pre>.....FF*..

PENDING:
 Boxer is pending - ./boxer_spec.rb:90
FAILING
  Boxer first that fails - ./boxer_spec.rb:82
    expected: false
     got: true
    (compared using ==)

  Boxer second that fails - ./boxer_spec.rb:86
    expected: false
     got: true
    (compared using ==)

Finished in 0.02068 seconds.</pre>

And voila, the formatter is working as supposed. Or, is it? :) Lets add some colors! Adding colors is really easy, we just need to require the [ConsoleCodes](http://www.rubydoc.info/gems/rspec-core/RSpec/Core/Formatters/ConsoleCodes) module. The ConsoleCodes module provides helpers for formatting console output with ANSI codes, for example colors and bold. So, the final version of our GroupingFormatter is: 

{% gist ac1a1fab4506ed9c4166 %}

As you can see, we are using the *ConsoleCodes.wrap* method which wraps a piece of text in ANSI codes with the supplied code in the arguments. You can now test our new colored formatter:


<pre>rspec some_file.rb --require ./grouping_formatter.rb --format GroupingFormatter</pre>

## Using your new GroupingFormatter

Our formatter is now working, but how can we put it to use? One way to use it is by running the specs and requiring the formatter in the RSpec command:

<pre>rspec some_spec.rb --color --require ./grouping_formatter.rb --format GroupingFormatter</pre>

This works alright. But, requiring your formatter every time you run your specs is boring.

### Meet .rspec

RSpec's documentation says that RSpec reads command line configuration options from files in three different locations:

*   Local: `./.rspec-local - `This file should exist in the project's root directory. It can contain some private customizations (in the scope of the project) of RSpec and should be ignored by git.
*   Project: `./.rspec`  - This file should exist in the project's root directory. It usually contains public project-wide customizations of RSpec and is usually checked into the git repo.
*   Global: `~/.rspec` - This file exists in the user's home directory. It can contain some personal customizations of your RSpec and is applied to every project where RSpec is used on your machine.

So, we can add a _.rspec_ file in our project's folder with the following contents:

<pre>--color
--require ./grouping_formatter.rb
--format GroupingFormatter</pre>

RSpec will read this file every time we run our specs, so this means that we can run our specs without specifying these options in the rspec command:

<pre>rspec some_spec.rb</pre>

This will now work with our new formatter.

### Using it in a Rails app

Lets integrate our new formatter in a Rails application. Using the formatter in your Rails application is done in two steps:

1.  The formatter class must either be in Rails' autoload path, or manually required in the _spec_helper_. My personal preference is to require it manually because it's more verbose.
2.  In the **RSpec.configure** block in the _spec_helper_, you need to register the formatter to RSpec. This is done by:

    {% highlight ruby %}
    config.formatter = NameOfTheClass
    {% endhighlight %}

or, in our case:

    {% highlight ruby %}
    config.formatter = GroupingFormatter
    {% endhighlight %}

That's it. Now when you run your specs the new formatter will be used by RSpec.

## Outro

I hope you found this (quite long) post informative and interesting. If any of you wrote your own RSpec formatters, please, share them with me and the others in the comments - I am very curious to see what you've come up with. Thanks for reading to the very end!
