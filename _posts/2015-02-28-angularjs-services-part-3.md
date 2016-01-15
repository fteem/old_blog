---
layout: post
title: "AngularJS Services Part 3: Service"
category: articles
tags: [angularjs, google, javascript, service]
---

So, Service is basically the same as Factory, it just has one key difference.
Service treats the function as a constructor, meaning, the service will call
**new** on the function and return the resulting object as a service object.
Oh, if you haven't yet read about Factory, check out
[AngularJS Services Part 2: Factory](/angularjs-services-part-2).

## Using Service

Declaring a Service is easy. Lets look at an example.

{% gist ff111cacb8f7959d103b %}

As you can see, we create a named function GreetingService and we create a
service out of it. One slight difference is that we are treating the function as
a constructor. This means that, like I said before, the Service will call
**new GreetingService()** when instantiating the service singleton.

In case you are wondering, in the example, we can use the GreetingService prototype too.

{% gist 59a79d73a3a453a411b8 %}

These examples are basically the same. One advantage of Prototypes is that it
allows us to use prototype-based inheritance.

## Service in the wild

I will use the same example from [my last post about Factory](/angularjs-services-part-2).

{% gist f0b64471febfc9574e18 %}

We use [OpenWeatherMap's API](http://openweathermap.org/) and we'll fetch
London's (UK) current temperature using a service.

Angular's built-in **$http** service is injected into our service so we can
make HTTP calls to a external API.

The rest, is simple - we issue an HTTP call, use promises to handle successful or
errored requests and we return back the promise itself.

What would injecting the service in a controller look like?

{% gist 5e8ed5cee3504f39f548 %}

Easy! The service is injected in the controller, get() is called on the
service and the returned promise from the service is handled. Then, when we get
the temperature in the promise we can do whatever we want with it.

As you can see, services are pretty simple compared to Provier or Factory. Although simple,
we can use protype-based inheritance. What does that mean?

## Inheritance via prototype chains

If you are unfamiliar with this topic, I recommend you read
[MDN's article on this topic](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
before continuing with the example.

So, since Service allows us to use prototypes, let's see how we can leverage this.

{% gist adead50ea2efbe2bcd63 %}

You can see in the code above how we can use inheritance to our own benefit here.
I am guessing you already know that GreetingService will have both, the
hello() and bye() functions. Also, what's cool about this approach is that we can
make Greet a service as well. These are some nice ways you can use the Service service in
AngularJS. Although it provides a lot of flexibility, can be a bit confusing.

In the next post we will look at the last two types of services: Constants and Values.
