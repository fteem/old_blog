---
layout: post
title: "AngularJS Services Part 1: Provider"
tags: [angularjs, google, javascript]
---

I started using AngularJS couple of months ago, when [we](http://siyelo.com#team)
got a new client that wanted us to help with building an app written in Angular.

After couple of months of using it, I found myself struggling with uber-phat controllers and
I started thinking of solutions about extracting knowledge out of the controller into
separate entities. Also, having the ability to easily inject those entities back
into the controller is **a must**.

When I found out about services it really reminded me of service/utility classes
that I was used to write in Ruby. Interestingly, the people at Google added 5 types
of services in Angular - to fit all your needs.

* Provider
* Constant
* Value
* Factory
* Service

Problem is, this was quite confusing to me, and I guess for any dev that just got
started with AngularJS. So I am writing this blog post that's part of my learning
and understanding of AngularJS services, and also, to help other fellow AngularJS
developers to understand services.


## Overview

Taken from AngularJS' [documentation on services](https://docs.angularjs.org/api/auto/service/$provide):

<blockquote>
An Angular service is a singleton object created by a service factory.
These service factories are functions which, in turn, are created by a service provider.
The service providers are constructor functions. When instantiated they must contain
a property called $get, which holds the service factory function.
<br/>
<br/>

When you request a service, the $injector is responsible for finding the correct
service provider, instantiating it and then calling its $get service factory function
to get the instance of the service.
</blockquote>

Mildly confusing. Okay, so, what is important:

1. Every service in Angular is a [singleton](https://en.wikipedia.org/wiki/Singleton_pattern).
2. Every service in Angular has a factory.
3. These factories are functions, that are created by the service providers.
4. A servide provider contains the constructor function of the factory. This constructor function is **always called $get**.

What happens when you inject your newly created service to (let's say) a controller:

1. the AngularJS injector finds the provider and instantiates it
2. then it calls the $get function on the provider instance
3. $get returns a service instance
4. Voila! The service is injected in your controller!

## Example (using Provider)

Let's take a look at a tiny example. As we all know, the internet is made for cats.
So, say we have a Cat provider.

{% gist 96e0c13497763581893c %}

As you can see, this provider has a $get function that will return an instance of the
service, which will be an object literal, containing two properties - the name and the color
of the cat.

We can now inject this service in a controller and use it in a template.

{% gist f5ef1ee7331490937a3c %}

{% gist 8a33a765ca00732d21b5 %}

This is a very simple example of how to create services using the **provider** built-in service.


## Provider

So we saw how we can easily use **provider**. Let's see what else provider provides (no pun intended!).

Every provider we create (i.e. see first code snippet) is a module that we can inject.
What's cool about Angular is that we can configure any injectables before they are being
injected (i.e. in a controller). Basically, the whole point of **provider** is to enable the
developer **to configure the service provider before it creates the service**, therefore
making services much more flexible.

Take this for example:

{% gist eb506656db18c6b13e3f %}

Let me walk you through this (bear with me!).

This is a Cat service provider, which contains the $get factory function that will return the
actual service object. The object, as you can see on lines 17-21, will contain the
name, color and age properties. The name and color are hardcoded, meaning, they never change.

I mean, who dyes their cat's fur or changes that little cute creature's name? Right?

But, like any other cat, it ages. So we have a hardcoded date of birth and a **setAge** method.
Take note that the **setAge** method, as well as the **calculateAge** are bound/in the scope of
the provider, not the factory function.

The cool thing happens on lines 40-42. The .config block is get executed during
the provider registrations and configuration phase. This means that we can do any
custom configuration to the provider before we **actually use** the service object that
the factory method will return.

Take note that on line 40, we pass in the **catProvider** to the config block,
because it's the provider that we are configuring, not the service. The service
will be an object that the $get function will return.

In our example, the config block calls **setAge** which then calls **calculateAge** which
dynamically calculates the age of the cat.

If one wants to set the age of the cat manually, he can pass a number as a parameter to the
setAge function.

Also, if one completely removes the configuration block, the age of the cat
will be set to 1. I know it doesn't really make sense, but for this example's sake lets leave it like that.


## Provider (and config) in the wild

Take for example [Restangular](https://github.com/mgonto/restangular). For those who haven't heard of it
Restnagular is basically a AngularJS service that provides a wrapper on top of $http and
is made to handle Restful Resources easily.

In the documentation, in the section [How to configure them globally](https://github.com/mgonto/restangular#how-to-configure-them-globally)
the author wrote a [very cool snippet](https://github.com/mgonto/restangular#configuring-in-the-config) showing
how one can configure the RestangularProvider **before** any Restangular service is created.

For example:

{% gist 5b6bebcf18b52b460456 %}

Here, we tell the RestangularProvider that every remote resource that we try to fetch (i.e. /users )
should have a base URL of **/api/v1**. So, when Restangular fetches the users, it will hit **/api/v1/users**.

This is just a small and tiny example of the flexibillity that AngularJS Providers gives us the developers.


