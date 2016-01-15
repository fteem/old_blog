---
layout: post
title: "Exception handling and testing it with Minitest"
tags: [ruby, testing, tdd, exceptions, errors]
---

When testing our code, we usually go for the happy path (TM). We are awesome developers, 
we test our code, we are careful and there's no way our code might crash. Or not really?
 I often try to think of software as a live being. It thinks, it does stuff and sometimes 
it gets some things wrong. Just like us. We sometimes trip up while walking, we drop our 
keys or forget them on our desk at the office. It's normal. So, how can we test our code 
for these rare occurances? 

## What is what?

Errors, exceptions and failures. It's really hard to tell them apart. More so, it's
harder to know when to use what. Avdi Grimm in his book "Exceptional Ruby" uses
Bertrand Meyer's definition of these words:

- **An exception** is the occurrence of an abnormal condition during the execution of a software element.
- **A failure** is the inability of a software element to satisfy its purpose.
- **An error** is the presence in the software of some element not satisfying its specification.

Also, he mentions:

<em>
  Note that failures cause exceptions... and are in general due to errors.
</em>

Give yourself time for this to sink in. 
I think that these definitions very much explain the difference between 
errors, exceptions and failures.

## Why testing them?

Let's start this with a hypotetical example. Say you are building a gem, that's 
basically an API wrapper.  The code has nice coverage percentage and you are confident 
that it works as it should. I guess you know where I am heading with this example. 
The obvious question is - what happens if for whatever reason, the API is not responsive? 
Maybe a switch in the datacentre died and they need couple of minutes to re-route 
the network. Or, maybe the API server crashed for whatever reason. What do we do than?

The problem with errors (which cause failures, whose product is exceptions) is that 
they are very often hard to think about. But they are real. Just like us, we never 
think about forgetting our keys on our desk, but, it happens. While in real life we can
pretty much always go back and get the keys from our desk, software isn't that intelligent by
default. It's our duty to make it intelligent, or in other words, we have to 
handle exceptions in our code.

So, why test them? Well, if you have error handling code, you should have tests. You 
should always aim for 100% code coverage. Simple as that. 

## Add error handling

I am the author of this tiny gem called [Forecastr](https://github.com/fteem/forecastr). 
Given that the gem doesn't know how to handle errors, let's add the code so it can 
handle errors and test it.


{% highlight ruby %}
require "net/http"
require 'json'

module Forecastr
  class Client
    class << self
      def search_by_city(city_name)
        uri = UriBuilder.by_city(city_name)
        response = Net::HTTP.get(uri)
        json = JSON.parse(response)
        Forecastr::DataContainer.new(json)
      end

      def search_by_coordinates(latitude, longitude)
        uri = UriBuilder.by_coordinates(latitude, longitude)
        response = Net::HTTP.get(uri)
        json = JSON.parse(response)
        Forecastr::DataContainer.new(json)
      end
    end

  end
end

{% endhighlight %}

This is the code that actually does the API calls. The first method, ```search_by_city``` accepts a city name as argument and issues the API call with the city name.
The second method, ```search_by_coordinates``` accepts the coordinates of
the location that we want to get the weather for. Just like the first method,
it issues the API call and parses the JSON response.

(As I sidenote - yes, I am aware that these methods are not as DRY as they should be, but lets stick to this code, at least for the purpose of this post.)

Now, after seeing this code, there's an obvious question - when it issues the GET 
request to the API, what would happen if the API is down? Or, if our internet
connection dies? 

I guess you can notice that there's a clear gap here. There's no code that can handle a HTTP 
timeout, or any other type of failure. 

There are couple of ways that we can do this. 

## Make some noise

When an exception happens, one option is to let it make noise. What I mean by that 
is just to let it explode.

{% highlight bash %}
client.rb:12:in `search_by_coordinates': execution expired (Timeout::Error)
{% endhighlight %}

Boom! And this is fine, but you want to have this covered by tests.

{% highlight ruby %}
  def test_raises_timeout_error
    stub_get("http://api.openweathermap.org/data/2.5/weather?lat=42.0&lon=21.4333").to_timeout
    assert_raises TimeoutError do
      Forecastr::Client.search_by_coordinates(42.0, 21.4333)
    end
  end
{% endhighlight %}

As you can see it the test above, the syntax to do this in Minitest is to use
the ```assert_raises``` method. It accepts an exception class as it's first 
parameter. Also, it expects that it's block will raise the exception specified as 
argument. 

## Make some noise, your way

Another option is to create your own exception classes. You can do this by 
subclassing an stdlib error, like TimeoutError.
 
{% highlight ruby %}
module Forecastr
  class RequestTimeout < ::TimeoutError; end

  class Client
    class << self
      def search_by_coordinates(lat, lon)
        uri = UriBuilder.by_coordinates(lat, lon)
        begin
          response = Net::HTTP.get(uri)
        rescue TimeoutError
          raise Forecastr::RequestTimeout.new("Request timeout.")
        end
        json = JSON.parse(response)
        Forecastr::DataContainer.new(json)
      end
    end

  end
end
{% endhighlight %}

Here, we ```rescue``` the TimeoutError that the GET request returns 
and we raise our own custom error.

Again, we can test this out by using the ```assert_raises``` assertion.

{% highlight ruby %}
  def test_raises_timeout_error
    stub_get("http://api.openweathermap.org/data/2.5/weather?lat=42.0&lon=21.4333").to_timeout
    assert_raises Forecastr::RequestTimeout do
      Forecastr::Client.search_by_coordinates(42.0, 21.4333)
    end
  end
{% endhighlight %}

This is particularly useful because when you know how the API works, you can
customize your errors to mimick the errors that the API can produce. 

## Benign values

This is also a very interesting approach to exception handling. 
Benign values mean that the program won't bomb out, but, it will return a 
meaningless value of some sort. 

What do I mean? Let's see an example:

{% highlight ruby %}
def search_by_city(city_name)
  uri = UriBuilder.by_city(city_name)
  begin
    response = Net::HTTP.get(uri)
  rescue TimeoutError
    response = ""
  end
  json = JSON.parse(response)
  Forecastr::DataContainer.new(json)
end
{% endhighlight %}

You see, instead of bombing out, we set the response to blank string. This will
not make the program stop, but, it will be obvious that the values we got back are
not right.

Testing this is pretty trivial:

{% highlight ruby %}
class Forecastr::ClientTest < Minitest::Test
  def test_handling_exceptions_with_benign_values
    stub_get("http://api.openweathermap.org/data/2.5/weather?lat=42.0&lon=21.4333").to_timeout
    result = Forecastr::Client.search_by_coordinates(42.0, 21.4333)
    assert_equal nil, result.temeprature
  end
end
{% endhighlight %}

Basically, our program didn't bomb out, but the forecast object got created 
without any real data in it. Usually, when doing this, it's best to log these events
so there's a written record that this occurred.

## Conclusion

While there are more tactics to handling exceptions in Ruby, it's very crucial we test them.
If we are not testing how our applications respond to errors, eventually, we will find out. 
The problem is that the experience will not be a pleasant one. Whichever way you prefer,
it's best to know how our application will handle the exceptions and (possibly)
recover from them.

