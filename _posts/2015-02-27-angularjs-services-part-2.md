---
layout: post
title: "AngularJS Services Part 2: Factory"
tags: [angularjs, google, javascript, factory]
---

If you don't know the basics of AngularJS service, I recommend you read the
other article I wrote on
[AngularJS Services Part 1: Provider](/angularjs-services-part-1).

## Provider v.s. Factory

Factory is a thin wrapper on top of Provider. While Provider provides us the
ability to configure the service provider before injection (creating the service object),
Factory lacks that ability.

Although it is short of this functionality, most of the Angular devs use factories
because they are, in my opinion, the easiest to use and really cover all the
general needs that one might need from a service. This does not mean that
Provider is bad - it is definitely great for certain use cases (configurability!).

That said, lets take a look at factories.

## Using Factory

Factories are defined very similarly as Providers. They return a value, a function
or an object literal. Usually developers go with returning an object, which
would result in a nice interface to the function. For example:

{% gist 1fffd9564af90513201c %}

Then, we can inject the GreetingService in a controller:

{% gist b3a1de38077520def849 %}

As you can see, returning an object from the service creates a nice looking
interface to the method.

## Factories in the wild

Usually, the practice is to use Factory (or services) when you need a reusable piece of code
that you can inject into a controller or another service.

Lets take a look at, a more real-life-like, example:

{% gist 316f95395518b2547952 %}

Here we are using [OpenWeatherMap's API](http://openweathermap.org/) which is
free to use. We will fetch London's (UK) current temperature using a service.

As you can see, Angular's built-in **$http** service is injected into our service
so we can issue HTTP calls to a remote API.

The API's url is defined as a variable and there is only one method in the service, get().
The url can also be extracted into another service i.e. constant and injected
back into the TemperatureService, but lets keep it simple until we get to constants,
in one of the next articles.

The rest, is simple - we issue an HTTP call, use promises to handle successful or
errored requests and we return back the promise itself.

What would injecting the service into a controller look like? Lets see:

{% gist 5e8ed5cee3504f39f548 %}

Simple, right? We inject the service in the controller, call get() on the
service and handle the returned promise from the service. Then, when we get
the temperature in the promise we can do whatever we want with it.

You can use these files to play with these services - it should all work well.
Just make sure you have an HTML file that loads AngularJS from Google's CDNs and
make sure you include these files after Angular is loaded.
