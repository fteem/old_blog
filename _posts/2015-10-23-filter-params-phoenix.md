---
layout: post
title: "Filter request params from Phoenix logs"
tags: [elixir, phoenix, logs, params]
---

Phoenix is a really powerful and customizable framework. One of it's small but important
configurations is filtering custom params from the logs. I am sure that this 
will be more interesting to beginner than experienced developers, but nevertheless, 
let's see what's the motivation behind this and how to do it in Phoenix.

## Motivation

First, I'd like you to understand the motivation behind this and why this is useful. 
Think about it. Our applications usually have login credentials, i.e. username and
password. In our databases, we almost always keep the username in plain text, while
the password is encryped with some sort of a hashing algorithm. 

While logging events is a very nice, although simple, feature of any web framework
(or, most of the software in the world), it can be a security flaw if not 
configured properly. Logs enable us, the people that build and maintain the software, 
to see how our application is behaving. It's logging all sorts of information. All
of the logs are stored on the hard drive of the server that's running the application.

So, it we are storing the passwords in the databases as hashes, why would we want
the logs to contain any login credentials in plain text? This is quite a security flaw. 
If someone gains access to the logs, they'll have access to a lot of user credentials.

## In practice

All of the web frameworks (that I've seen/tried/used) filter the `password` and 
`password_confirmation` params in their logs. But, what if you are building an API? 
For example, what if the user needs to supply an `API_KEY` param with every request? 
We wouldn't want the key to be stored in the logs in plain text.

Custom params filters come into play here. Phoenix allows us to customize the
list of filtered params from the logs. This is easily achived by adding:

{% highlight elixir %}
config :phoenix, :filter_parameters, ["password", "api_key"]
{% endhighlight %}

Now, there are two things to keep in mind. The first is to decide in which environment(s) 
we want the filtering to be applied. For example, if we want this to happen only in
production environment, this line needs to be added in the `config/prod.exs` file.
In case we need filtering in all environments, it should be added in the `config/config.exs`
file.

The second thing is to understand that by adding this line, the default filters 
are overriden. This means that if we leave out `"password"` from the list, 
it will not be filtered, although it's a default configuration.

Hope this clears up filtering params from the logs in Phoenix. If you have any 
questions or feedback, feel free to join the discussion below!


