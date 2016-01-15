---
layout: post
title: "Rack: Writing middleware"
tags: [ruby, rack, middleware]
---

Last time I wrote about [the basics of Rack]({{site.url}}/rack-first-principles){:target="_blank"} and
writing a tiny Rack application. If you are unsure what Rack is and what is it's purpose,
I recommend you read the other post, famirialize yourself with Rack and get back to this
post. If you think you know enough about Rack, please, carry on reading.

## Enter: Middleware

So, middleware. Lets take it from the basics. What is middleware? Remember that
Rack "wraps" HTTP requests and responses? Also, remember that it has this interface
(or API) which allows you to play with the request and the responses?

Well, middleware is a term that is used when speaking for a piece of software which
is somehow related to the execution of the web application, but is not directly involved with
the execution of the requested task.

Confusing? Okay, lets put it in another way.

Think of logging. You request a resource from an API, and the API responds with your results.
Although you did not request the API to log your request (why would you?), the API has
a logger which records everything that goes in and out of the application. So you see,
although the logger is not directly involved with handling your request, the logger is
middleware that writes down what you requested and what the API responded with.

## How it works

Every Rack application is a class. The same applies to Rack middleware. Lets write
the class and see some basic rules that apply to writing middleware classes.

{% highlight ruby %}
class MyMiddleware
  def initialize(app)
    @app = app
  end
end
{% endhighlight %}

When writing a Rack middleware class, the first argument of the ```initialize``` method
is the application, or the request handler. If that's confusing, think of it in this way
- the request handler has to be passed in the middleware class, so the middleware can wrap
the execution of the application and do something to the request and the response of
the application.

Another rule that applies to a middleware class is the ```call``` method. The
```call``` method executes the application which returns the status, the headers and the
body of the response.

Take this for an example:

{% highlight ruby %}
# test_handler.ru

class MiddlewareTwo
  def initialize(app)
    @app = app
  end

  def call(env)
    puts "MiddlewareTwo reporting in!"
    puts "The app is: #{@app}"
    puts "The has the methods: #{@app.methods - Object.methods}"
    status, headers, body = @app.call(env)
    [status, headers, body]
  end
end

class MiddlewareOne
  def initialize(app)
    @app = app
  end

  def call(env)
    puts "MiddlewareOne reporting in!"
    puts "The app is: #{@app}"
    puts "The has the methods: #{@app.methods - Object.methods}"
    status, headers, body = @app.call(env)
    [status, headers, body]
  end
end

class HandlerClass
  def self.call(env)
    puts 'Handling the request...'
    [200, { "Content-Type" => "text/html" }, ["<b>Request handled.</b>"]]
  end
end

use MiddlewareTwo
use MiddlewareOne
run HandlerClass
{% endhighlight %}

When we run this file with ```rackup test_handler.ru```, the WEBrick server will boot
and our request handler will be served (your output may vary):

{% highlight bash %}
➜  rackup ~/Desktop/test.ru
>> Thin web server (v1.5.1 codename Straight Razor)
>> Maximum connections set to 1024
>> Listening on localhost:9292, CTRL+C to stop
{% endhighlight %}

When you visit localhost:9292 in your browser (or cURL it), you will see the "Request handled."
message. But the server logs are what we are interested in:

{% highlight bash %}
MiddlewareTwo reporting in!
The app is: #<MiddlewareOne:0x007fbcdb111f08>
The has the methods: [:call]
MiddlewareOne reporting in!
The app is: HandlerClass
The has the methods: [:call]
Handling the request...
{% endhighlight %}

As you can see, the app is actually the handler, or, the ```HandlerClass```. It only has one method, the
call method. But, what's really interesting is the order in which the statements are executed.
First, ```MiddlewareTwo``` wraps the request and prints the app and it's methods. After,
it executes the ```@app.call(env)``` line, which is the ```MiddlewareOne``` class.
When it calls the ```call``` method, it actually executes the ```MiddlewareOne``` call method.
Then, that method prints out the app and it's methods. Here, the app is the ```RequestHandler```.
After it calls the ```call``` method, the response is served.

This is what the wrapping looks like:

{% highlight ruby %}
MiddlewareTwo[ MiddlewareOne [ RequestHandler ] ]
{% endhighlight %}

This means that ```MiddlewareTwo``` executes ```MiddlewareOne``` which executes ```RequestHandler```.

## Loggster

Let's take it to the next level and write our own logging middleware. We'll call the class Loggster! :-)
What we are aiming for is middleware, that will wrap our Rack application and will log the
requests that come in to a logfile. Sounds simple? Lets see...

{% highlight ruby %}
# loggster.ru
class Loggster
  def initialize app
    @app = app
  end

  def call env
    start_time = Time.now
    status, headers, body = @app.call env
    end_time = Time.now

    Dir.mkdir('logs') unless File.directory?('logs')
    File.open('logs/server.log', 'a+') do |f|
      f.write("[#{Time.now}] \"#{env['REQUEST_METHOD']} #{env['PATH_INFO']}\" #{status} Delta: #{end_time - start_time}s \n")
    end

    [status, headers, body]
  end
end

class RackApp
  def self.call env
    [200, {'Content-Type' => 'text/html'}, ['Hi!']]
  end
end

use Loggster
run RackApp
{% endhighlight %}

The Rack application, or, the ```RackApp``` class just returns a HTTP 200 with a "Hi!" message. Simple stuff.
So, what does Loggster do? When we add the ```use Loggster``` line in the file, we tell Rack to wrap 
the request handler a.k.a. ```RackApp``` with ```Loggster```. This means that ```Loggster#call``` will 
call ```RackApp.call```, it will write to the log file and return the full response that ```RackApp``` returned.
The "magic" call method, is really simple. It checks if there's a directory called 
"logs" and creates it if it does not exist.  After, it creates a ```server.log``` 
file and inside, it writes the current time, the request method (i.e. GET), the 
URL that the web client requested and the response HTTP status.

If we run this file with ```rackup loggster.ru``` and visit the URL, we'll see that a directory called 
"logs" and a file "server.log" inside, have been created. So, how does the output of our simple logger looks?

{% highlight bash %}
➜ cat logs/server.log
[2015-06-27 01:24:37 +0200] "GET /something" 200 Delta: 1.3s
[2015-06-27 01:24:37 +0200] "GET /favicon.ico" 200 Delta: 0.3s
[2015-06-27 01:24:37 +0200] "GET /favicon.ico" 200 Delta: 0.2s
{% endhighlight %}

As you can see, I requested ```localhost:<port-number>/something``` via my browser. The browser requested the path 
and it also requested the favicon twice.

## Outro

As you can see, writing Rack middleware can be pretty simple. Of course, it all
depends on what functionality you would like the middleware to have. That being said,
the rules (or constraints) that Rack imposes when writing middleware are tiny and very clear.
In the next post, we will see how we can implement Rack middleware and mount it in
a Rails application. In the mean time, did you implement any Rack middleware yourself? 

Feel free to share your stuff (or ask for help, if needed) with me and my readers in the comments. 

