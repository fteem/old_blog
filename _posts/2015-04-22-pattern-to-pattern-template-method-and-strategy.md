---
layout: post
title: "Pattern to pattern: Template Method & Strategy"
tags: [angularjs, google, javascript]
---

Recently I wrote about the [Template Method pattern](http://rubylogs.com/template-method-pattern-in-ruby/ "Template Method Pattern in Ruby") and how it's implemented in Ruby. In the comments, one of the readers commented that the Template Method pattern is in fact the Strategy pattern. After thinking hard about how I should answer the question, I thought about writing a post comparing the two patterns. So, here it is - my version of design patterns head to head. Let's see what these two patterns have in common and what are their key differences.

### Template Method pattern

The most important thing to remember about this pattern is: boilerplate code goes in the parent class while the algorithm-specific methods are implemented (usually overridden) in the child classes. This means, that all of the child classes will share the functionality of the parent class (inheritance, duh!), but the parent class will only be a skeletal class, i.e. one cannot create objects out of it. On the other hand, the child classes will only override the methods that are specific to their domain. You can read more about this in [my recent blog post about the Template Method pattern](http://rubylogs.com/template-method-pattern-in-ruby/).

### Strategy Pattern

In it's core the Strategy Pattern (or SP) is about encapsulating logic into objects, making them **interchangeable** and using those objects as part of an algorithm. This means that an algorithm (context) can have it's behaviour changed **in runtime** by an object (strategy).

#### Implementing the Strategy pattern

Let's see a short example of how we can use the Strategy pattern. Let's create a simple _Invoice_ class.

{% gist 48a0c4a6272be7bed305 %}

As you can see, the Invoice is consisted of an amount, the name of the buyer and the name of the seller.  In it's constructor, a formatter of the Invoice is created. The formatter is an object of a class that has a method **format!** which takes the buyer, seller and amount as params. When the method is called, it will create the invoice in it's specific format. One very important aspect of the strategy classes is that the **context class expects the strategy classes to implement**, in our example,** a format! method**. For example, in Java this is achieved by using interfaces, which force a class to implement certain methods. But because Ruby is a dynamic language, there's no such thing as an interface and we must think about this aspect in advance. That being said, lets take a look at the formatter classes - they're all pretty simple.

{% gist f3a65e8736bbff541a98 %}

The _JSONFormatter_ creates the invoice in JSON format. As a side note, I am using the percentage string notation because it's a bit more readable this way.

{% gist e89d4878187c787f4778 %}

The _XMLFormatter_ creates the invoice in XML format.

{% gist 97a5e26d0bcd9c72422a %}

And, last but not least, _YAMLFormatter_ creates the invoice in YAML format. The beauty of the Strategy Pattern lies in the context (the _Invoice_) and the strategies (the formatters). You see, when we first create the _Invoice_, we can use it to print out a JSON formatted invoice. And it's cool. But the cooler thing is that you can **change how the _Invoice_ will behave in runtime**, meaning, you can **change the formatter object and the behaviour of the invoice will change**. Or, said in a more abstract way, **the context's behaviour can change with every different strategy you apply to the context**. So, how this applies to our code?

{% gist 5dab2c372bb63f0639a2 %}

The first time we generate the Invoice, we use the JSONFormatter and of course we get an invoice back in a JSON format.

{% gist e53a61f6f8dda4c96fe1 %}

Now, if we **change the formatter to that same Invoice object**...

{% gist 6c71c0a847e0323cb63f %}

...we will get the same invoice, formatted in XML.

{% gist ea535c0fb249014f5c5e %}

You see, our formatters change the behaviour of our _Invoice_.

### Strategy and Template Method

So what are the similarities and what are the key differences in these two design patterns? Well, it might be obvious to some of you, confusing to others (as it was for me at the beginning). Both patterns are about encapsulating domain-specific algorithms into objects. But, the key difference is that Strategy Pattern is about modifying a behaviour of a context in runtime using strategies, while Template Method Pattern is about following a skeleton implementation of an algorithm and modifying its behaviour by overriding methods of the skeleton class in the subclasses.
