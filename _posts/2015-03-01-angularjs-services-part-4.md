---
layout: post
title: "AngularJS Services Part 4: Value and Constant"
category: articles
tags: [angularjs, google, javascript, service]
---

So far we saw the magic of creating AngularJS services using
[Provider](/angularjs-services-part-1),
[Factory](/angularjs-services-part-2) and
[Service](/angularjs-services-part-3).
In this post, we will look at two more types of services - Value and Constant.

## Value

The Value service is basically a service that returns a single value, like,
string, object, number or an array. For instance:

{% gist 989fb78bfb8a184ee0b7 %}

The Value service is basically like writing a service using Provider, whose **$get**
function returns a plain value (string/object/number/array).

{% gist 25471b6fef5e051a66cb %}

If you are unfamiliar of the way Provider works, take a look [here](/angularjs-services-part-1).
One drawback of Value is that it cannot be injected into a module configuration
function, unlike Provider. On the other hand, it can be decorated using an Angular decorator.

## Constant

The Constant service is really the same like Value, with two key differences:

1. It **can be injected** into a module configuration function, like Provider.
2. A Constant service **cannot** be decorated using an Angular decorator.

For consistency's sake, lets see how constants are defined:

{% gist 09f124345e25202b4638 %}

One last note - injecting Value and Constant services in controllers is done just like
any other services.
