---
layout: post
title: "Understanding Reactive Programming with RxRuby"
tags: [ruby, reactive programming, rx]
---

Reactive Programming is a relatively new and interesting programming paradigm
that has picked up quite a bit of popularity lately. Out of curiosity, I did a
bit of reasearch over the weekend. In this blog post I will summarize what I
learned and try to explain what RP to any novice out there. Also, I show you how
to use the Reactive Extensions for Ruby. Let's dive in!

## The Motivation

I rarely have strong opinions, but I really think that to understand anything in
life, you need to understand the motivation behind it. In this case, to
understand what Reactive Programming is, we need to see what problems it is
solving.

All of the programming paradigms that we know of at this point of time are
invented more than 20 years ago. Yes, even functional programming. In fact, it
was invented like 60 years ago. The problem is, just like with anything that
evolves, priorities change.

In our case, the internet happend. Sure, +20 years ago the internet did exist,
but it had less users then the count of views that a popular video on YouTube
has today. Going forward 15-10 years ago, the internet grew enough, so we felt
the need for a ton of data processing and asynchronous behaviour.

Applications needs have changed, from poll-based to full reactive applications,
where the data is pushed to us. Each time, we're adding more complexity, more data,
and asynchronous behavior to our applications. So, reactive programming is a
way to manage all of that complexity, mindset switch included.

## Composition

The word invented might hurt someone's ears. At the low level, the paradigm is
basically a construct of two design patterns that have been around for more then
20 years. Actually, one of the most popular design patterns book, the Gang of
Four book, covers these two design patterns: the Iterable and the `Observer`
patterns. These two patterns are behavioral design patterns, which characterize
the interaction and responsibility of objects and classes.

### Iterator

Taken from the GoF book:

> The key idea in this pattern (the Iterator) is to take the responsibility for
> access and traversal out of the list object and put it into an iterator object.
> The Iterator class defines an interface for accessing the list's elements. An
> iterator object is responsible for keeping track of the current element; that
> is, it knows which elements have been traversed already.

Sounds familiar? I am sure it does. Ruby's `Enumerable` module does just that,
it returns an enumerator, which encapsulates the data and the traversal methods.
Okay, let's keep this short then - an iterator in Ruby is an instance of a class
that includes the `Enumerable` module. Which, it's pretty much any collection.

### Observer

The observer pattern is a design pattern in which an object, called
the subject, maintains a list of its dependents, called observers, and notifies
them automatically of any state changes, usually by calling one of their
methods. Usually, it's used to implement distributed event handling systems.

Basically, the subjects are the objects that send the messages to the objects
that observe the subjects (damn!). The subjects push the messages to the
observers, which is why this is called the Observer Pattern.

Ruby ships with the
[Observable module](http://ruby-doc.org/stdlib-2.3.0/libdoc/observer/rdoc/Observable.html)
in it's standard library, which provides a simple mechanism for one object
(subject) to inform a set of subscribers (observers) of any state change.

## Reactive Programming

Now that we have an understanding of both, the Iterator and the Observer
patterns, it's time for the Reactive Programming paradigm.

Accoring to the [Reactive Manifesto](http://www.reactivemanifesto.org/),
Reactive Systems are: Responsive, Resilient, Elastic and Message Driven. I don't
know about you, but these type of definitions confuse me a lot. In the way that
I understand the Reactive Manifesto, Reactive Systems are asynchronous, fault
tolerant, scalable and they communicate with non-blocking message-passing.

But, what interests us is the implementation, especially in Ruby. Would you be
surprised if I tell you that on the surface, the reactive programming paradigm
is basically an abstraction on top of the the `Observer` pattern in combination
with the `Iterator` pattern?

Don't believe me? Let's see a few examples.

## Reactive Extensions

From the [README](https://github.com/ReactiveX/RxRuby/blob/master/readme.md):

> The Reactive Extensions for Ruby (or, RxRuby) is a set of libraries to compose
> asynchronous and event-based programs using observable collections and
> Enumerable module style composition in Ruby.

There, told ya? `RxRuby` is a library, which does all of the abstractions for us,
so we can easily start with Reactive Programming in Ruby. Neat, eh?

### Observables

Let's see how `RxRuby` works, in it's simplest form.

{% highlight ruby %}
RxRuby::Observable.just(42)
{% endhighlight %}

A `RxRuby::Observable` is just a stream. A stream is a subject (or, an object)
that you can subscribe to (or, observe). Now, the `RxRuby:Observable` module
itself doesn't do anything, unless we invoke a method on it. Let's carry on
with the example:

{% highlight ruby %}
stream = RxRuby::Observable.just(42)
stream.subscribe {|num| puts "We received: #{num}" }
{% endhighlight %}

What do you think will happen here? Let's break it down. The object that we
receive is a just the number 42, wrapped as an observable. Since the observable
implements (most of) the `Enumerable` methods, we can do anything with it. But,
to avoid any complication in this example, we will just `subscribe` to the
observable and pass in a lambda. The lambda will be invoked on everytime the
stream pushes any data (or, in this case, just once).

The output of the example will be:

{% highlight bash %}
We received 42.
{% endhighlight %}

### Observing on an array

Now, since we have a lot of the methods from the `Enumerable` mixin available,
let's do something with an array. There's two ways of working with arrays in
`RxRuby`: via ranges and via plain arrays.

Via ranges:

{% highlight ruby %}
RxRuby::Observable.range(1,10)
  .select {|num| num.even? }
  .sum
  .subscribe {|s| puts "The sum of the even numbers from 1 to 10 is: #{s}" }
{% endhighlight %}

This will return:
{% highlight bash %}
The sum of the even numbers from 1 to 10 is: 30
{% endhighlight %}

Let's break it down. First, we create an observable range, with the numbers from
1 to 10. Then, we invoke `select` on the observable, with a lambda as a parameter.
The lambda takes each number of the range as a parameter, and filters all even
numbers from the observable, returning a new observable. Then, we invoke `sum`
on it, which sums all of the even numbers.

At the end, we `subscribe` on the observable that `sum` returns. By subscribing
we basically "watch over" the observable (or, the stream of data) and invoke the
lambda that we pass as an argument to the `subscribe`. Any time the observable
pushes any data, it will go through this "funnel" and invoke the last lambda at
the end, which will print out the message.  the observables push any data we

Doing this via arrays is very similar:

{% highlight ruby %}
arr = Array(1..10)
RxRuby::Observable.from_array(arr)
  .select {|num| num.even? }
  .sum
  .subscribe {|s| puts "The sum of the even numbers from 1 to 10 is: #{s}" }
{% endhighlight %}

This snippet will also return the same message at the end. The only difference
here is that we create an observable from an array.

## More RxRuby

The documentation on RxRuby is still scarse, but fortunately for us there are
a lot of [examples](https://github.com/ReactiveX/RxRuby/tree/master/examples) in
the Github repo.

Also, I have ported the
[RxJS Koans](https://github.com/Reactive-Extensions/RxJSKoans) to Ruby, for
easier learning of the library.
[RxRuby Koans](https://github.com/fteem/rxrubykoans) is still a bit unstable,
but it should give you a nice overview RxRuby's methods/objects.

## The future

I am not sure what the future holds for RxRuby, but it seems that reactive
programming is here to stay. Netflix has built quite a big architecture built
on the Reactive Systems paradigm. Also, the Reactive JavaScript community is
**super vibrant**, unlike the Ruby community. But, let's hope for the best!

As for me - I am looking for anyone that's profficient with RxRuby, to review
my RxRuby Koans, so we can get it moving somewhere.

## Notes

Some links I used while writing this article, that you might find interesting:

- [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
- [Overview of Reactive Programming](https://hackhands.com/overview-of-reactive-programming/)
- [What is Reactive Programming?](https://medium.com/reactive-programming/what-is-reactive-programming-bc9fa7f4a7fc)
- [The 7 Ways to Wash Dishes and the Case for Message-driven Reactive Systems](https://www.typesafe.com/blog/7-ways-washing-dishes-and-message-driven-reactive-systems)
- [RxRuby](https://github.com/ReactiveX/RxRuby)
- [RxJSKoans](https://github.com/Reactive-Extensions/RxJSKoans)
