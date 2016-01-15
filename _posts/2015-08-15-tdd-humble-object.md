---
layout: post
title: "TDD Patterns: Humble Object"
tags: [ruby, tdd, humble, object]
---

We all know that there are different design patterns. They are all quite trivial 
to learn, but, the trick lies in applying them. When should we use this or that 
pattern and will that help in making our code better and cleaner. Well, tests
are code as well and, you guessed it, there are some testing patterns that are 
around for a while. 

Today, we will take a look at one of them. It is a useful one and also it has an 
interesting name - the Humble Object Pattern.

## The problem

Think about this. You are building an API client/wrapper. You read the documentation,
you understand the model and the intent of the API and how everything is composed.
You start writing your code, keeping it in small chunks (i.e. classes). You are a
responsible programmer, you want to have your code well documented and tested. All
of this is nice. And then, you get to this part where you start mocking your HTTP calls
and stuff easily gets messy. 

How can we avoid this? Is there a better way?

## Humble Object

Let us take a detour. A bit of history first. I think that Uncle Bob has a really 
good definition of the Humble Object pattern in [Episode 23](http://cleancoders.com/episode/clean-code-episode-23-p2/show) 
of his Clean Code videos:

<blockquote>
  This pattern is applied at the boundaries of the system, where things are often
  difficult to test, in order to make them more testable. We accomplish the pattern
  by reducing the logic close to the boundary, making the code close to the boundary
  so humble that it doesn't need to be tested. The exctacted logic is moved into 
  another class, decoupled from the boundary which makes it testable.
  <br/> <br/>
  - Robert C. Martin
</blockquote>


In case you find the definition confusing, let's tear it apart. First, the boundaries. 
When one says boundaries, it means that the person is referring to the part of the
system that communicates with other software that is not written by you, but your 
software is dependent on it. For example, let's say your software creates cron jobs. 
The boundary lies beween the software that will call the cronjob system command and 
the operating system.

So, this means when using the Humble Object pattern, we extract as much logic as 
we can from the boundary class(es), thus making them humble. The extracted logic 
will be moved to another class, which will be easily testable. On the other hand,
the humble code will be dependant on the extracted class, but testing it won't be 
necessary, because it doesn't hold any business logic.

Usually, when explaining the humble object, people use GUIs or async code as examples.
We won't go down that path today. Let's try finding an example which we might run
into more frequently.

## Back to the problem

Now, that we undestand the motivation and the theory behind this pattern, let's 
continue with aforementioned design problem.

Tackling any programming problem, in my opinion, is best understood via some code. 
Here's an example. We will make a tiny API wrapper of the REST Countries API, more 
specifically, the [World Capitals API](https://restcountries.eu/rest/v1/capital/london).

{% highlight ruby %}
require 'net/http'
require 'json'

class CapitalsClient
  API_ENDPOINT = "https://restcountries.eu/rest/v1/capital/"

  def self.find(capital_name)
    uri = URI(API_ENDPOINT + capital_name)
    response = Net::HTTP.get(uri)
    result = JSON.parse(response).first
    currencies = result['currencies'].map {|currency| Currency.new(currency) }

    Country.new({ name: result["name"], 
                  capital: result["capital"], 
                  region: result["region"],
                  population: result["population"],
                  latitude: result["latlng"][0],
                  longitude: result["latlng"][1],
                  native_name: result["nativeName"],
                  currencies: currencies
    })
  end
end
{% endhighlight %}

The ```CapitalsClient``` will issue a GET request to the API endpoint, get the 
result and build a ```Country``` object from the results. This is what the JSON 
result looks like for London, UK:

{% highlight json %}
[
    {
        "name": "United Kingdom",
        "capital": "London",
        "altSpellings": [
            "GB",
            "UK",
            "Great Britain"
        ],
        "relevance": "2.5",
        "region": "Europe",
        "subregion": "Northern Europe",
        "translations": {
            "de": "Vereinigtes Königreich",
            "es": "Reino Unido",
            "fr": "Royaume-Uni",
            "ja": "イギリス",
            "it": "Regno Unito"
        },
        "population": 64105654,
        "latlng": [
            54,
            -2
        ],
        "demonym": "British",
        "area": 242900,
        "gini": 34,
        "timezones": [
            "UTC−08:00",
            "UTC−05:00",
            "UTC−04:00",
            "UTC−03:00",
            "UTC−02:00",
            "UTC",
            "UTC+01:00",
            "UTC+02:00",
            "UTC+06:00"
        ],
        "borders": [
            "IRL"
        ],
        "nativeName": "United Kingdom",
        "callingCodes": [
            "44"
        ],
        "topLevelDomain": [
            ".uk"
        ],
        "alpha2Code": "GB",
        "alpha3Code": "GBR",
        "currencies": [
            "GBP"
        ],
        "languages": [
            "en"
        ]
    }
]
{% endhighlight %}

For completeness sake, let's see the ```Currency``` and ```Country``` classes.

{% highlight ruby %}
class Currency
  def initialize code
    @code = code
  end

  def to_s
    @code
  end
end
{% endhighlight %}

{% highlight ruby %}
class Country
  attr_reader :name, :capital, :region, :population, :latitude, :longitude, 
    :native_name, :currencies

  def initialize attrs
    @name        =  attrs.fetch("name",nil)
    @capital     =  attrs.fetch("capital",nil)
    @region      =  attrs.fetch("region",nil)
    @population  =  attrs.fetch("population",nil)
    @latitude    =  attrs.fetch("latlng",[]).first
    @longitude   =  attrs.fetch("latlng",[]).last
    @native_name =  attrs.fetch("nativeName",nil) 
    @currencies  =  attrs.fetch("currencies", [])
  end
end

{% endhighlight %}

Now, let's revisit the ```CountryClient``` class. We've all done this - we fetch 
the JSON, parse it and build the currencies and country object(s). Now, testing is 
interesting. 

First, we'll need to stub the GET request using 
[Webmock](https://github.com/bblimke/webmock) and assert on the ```Country``` 
object that we receive as a result of the ```CapitalsClient::find``` method. 
Another approach is to use VCR and record the request going out and replay it 
whenever needed.

{% highlight ruby %}
require 'minitest/autorun'
require 'webmock/minitest'

class CountryClientTest < Minitest::Test
  def test_client_fetches_countries
    response = %Q{
      [
        {
          "name": "United Kingdom",
          "capital": "London",
          "region": "Europe",
          "population": 64105654,
          "latlng": [54, -2],
          "nativeName": "United Kingdom"
        }
      ]
    }
    stub_request(:get, CountryClient::API_ENDPOINT + "London").to_return(body: response)
    country = CapitalsClient.find("London")
    assert_equal "London", country.name
    assert_equal "United Kingdom", country.name
    assert_equal "Europe", country.region
  end
end
{% endhighlight %}

Now, this works, it's fine. But, what exactly are we testing here? I am sure we 
have all done this multiple times. Look at the test class name - ```CountryClientTest```.
We should be testing the client, not setting assertions on the country object. 
The ```CountryClient``` acts like a factory, not like an API client. 

Also, think about this - stubbing, although it looks fine, it's basically isolation. 
While we cannot completely rely on pulling real data from the API for each of our
tests, we shouldn't also go overboard with it. 

While one can argue that stubbing external services is all good, what would the 
case be if you used a library that was actually fetching the data and building 
it for you? What would you test if the wrapper was made by someone else and
you are using it in a Rails app? You could go on and stub the library, but you have
no idea if the API and/or the wrapper had any changes made to them.

But, let's take a step back. How can we apply the humble object pattern here? 

## Applying the pattern

If you remember, the pattern states that we need to extract most of the logic near
the boundaries of the system, so the code on the boundary itself is so humble, it 
doesn't need to be tested. But, the humble object should be **dependent** on the
extracted code.

Let's try to refactor our code by doing exactly that. 

First, the ```CountryClient```. It sure does more than it should. Let's make it humble.
The first step would be to make it an API client. Exactly that, nothing more or less. 
This means that it will only issue HTTP GET to the API. Hint: think of the 
Single-responsibility principle.

{% highlight ruby %}
require 'net/http'

class CapitalsClient
  API_ENDPOINT = "https://restcountries.eu/rest/v1/capital/"

  def self.find(capital_name)
    uri = URI(API_ENDPOINT + capital_name)
    Net::HTTP.get(uri)
  end
end

{% endhighlight %}

That's it. Again, ```CapitalsClient``` just sends the request and it returns it's 
response. Now, the extracted logic. 

Since the code that we extracted was actually building the ```Country``` object, 
we can create a ```CountryBuilder``` class out of it:

{% highlight ruby %}
require 'json'

class CountryBuilder
  def self.build json
    result = JSON.parse(response).first
    currencies = result['currencies'].map {|currency| Currency.new(currency) }

    Country.new({ name: result["name"], 
                  capital: result["capital"], 
                  region: result["region"],
                  population: result["population"],
                  latitude: result["latlng"][0],
                  longitude: result["latlng"][1],
                  native_name: result["nativeName"],
                  currencies: currencies
    })
  end
end
{% endhighlight %}

The ```CountryBuilder.build``` method will receive the JSON and build the ```Country```
object on it's own. The next, obvious step, is to test this class. If you look at
the test we wrote earlier, you can notice that the test is 90% done, it just needs
some tweaking. 

{% highlight ruby %}
require 'minitest/autorun'

class CountryBuilderTest < Minitest::Test
  def test_build_builds_the_country
    response = %Q{
      [
        {
          "name": "United Kingdom",
          "capital": "London",
          "region": "Europe",
          "population": 64105654,
          "latlng": [54, -2],
          "nativeName": "United Kingdom"
        }
      ]
    }
    country = CountryBuilder.build(response)

    # Sanity check
    assert_equal Country, country.class

    # Useful assertions
    assert_equal "London", country.name
    assert_equal "United Kingdom", country.name
    assert_equal "Europe", country.region
  end
end
{% endhighlight %}

The key differences in the new and old test is that the new one is missing the
Webmock request stub. Also, we are testing the builder class, which does what it should -
receives the response as a JSON, builds an object of a class and returns it. But, 
what happens now to the ```CapitalsClient```? Well, nothing too complicated. If 
you remember, the pattern states that the humble object should depend on the 
extracted code. 

If you look at the code below, it should all make sense:

{% highlight ruby %}
require 'net/http'

class CapitalsClient
  API_ENDPOINT = "https://restcountries.eu/rest/v1/capital/"

  def self.find(capital_name)
    uri = URI(API_ENDPOINT + capital_name)
    response = Net::HTTP.get(uri)
    CountryBuilder.build(response)
  end
end
{% endhighlight %}

So, we add the dependency, making ```CapitalsClient``` use ```CountryBuilder``` to
create the country out of the JSON payload. 

And, what about testing ```CapitalsClient```? Well, we don't really need to test it. 
Even if we wanted to test it, we could only write a test with a test spy that would 
expect ```CountryBuilder.build``` to be called. But, how useful is that test? 
If we wrote it, we would tie our test to the implementation of the production code. 
This means that if the implementation of this method changes in the future, but 
it's output does not, our tests will fail although we haven't broken our code.

## Outro

As you can see, although very simple, this humble pattern can make a difference 
when we want to leave out heavy stubbing to external services, APIs or interfaces 
and just focus on our code where "the magic" happens. Also, at least for me, this 
type of refactor comes very natural in these situations. But, knowing the pattern 
guides us towards making only one entry point to our code (or, dependency).

