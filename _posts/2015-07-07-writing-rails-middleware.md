---
layout: post
title: "How to write Rails middleware"
tags: [rails, rack, middleware]
---

In my last two posts about Rack, I wrote about [the basics of Rack]({{site.url}}/rack-first-principles){:target="_blank"} 
and [how to write middleware]({{site.url}}/writing-rack-middleware){:target="_blank"}. If you 
have no idea what this is about, I recommend reading the last two posts (in the order above). 
For the rest of you, carry on - today we will see how to write awesome Rails middleware 
and how to use it in any Rails application. Rails and Rack play together really nice, 
so keep on reading!

## Rails on Rack

Rails by default has bunch of middleware loaded that is crucial to Rails' working. If you want
to see what middleware your Rails app is using, open it up in command line and run:

{% highlight bash %}
rake middleware
{% endhighlight %}

You will see a big list of middleware classes that your current Rails app is using. 
This is a sample output on a app that I am working on. Keep in mind that your output 
may vary.

{% highlight bash %}
use ActionDispatch::Static
use Rack::Lock
use #<ActiveSupport::Cache::Strategy::LocalCache::Middleware:0x007f93ff6810d8>
use Rack::Runtime
use Rack::MethodOverride
use ActionDispatch::RequestId
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use ActionDispatch::DebugExceptions
use BetterErrors::Middleware
use ActionDispatch::RemoteIp
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::ConnectionAdapters::ConnectionManagement
use ActiveRecord::QueryCache
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use ActionDispatch::ParamsParser
use ActionDispatch::Head
use Rack::ConditionalGet
use Rack::ETag
use ActionDispatch::BestStandardsSupport
use Warden::Manager
run MyApplication::Application.routes
{% endhighlight %}

The list of middleware is quite big. If you are curious what any of these middleware classes do, 
check [this list out](http://guides.rubyonrails.org/rails_on_rack.html#internal-middleware-stack){:target="_blank"}.
Cool stuff, right? Lets see what rules and conventions apply to writing Rails middleware 
and how we can leverage those to write our own middleware.

## Setting up a middleware class

Just like any Rack middleware that we wrote before, we will need to write a middleware class. 
Since v3.2, Rails gave us the ability to add middleware classes to ```app/middleware```. 
So, lets add that class.

{% highlight ruby %}
# app/middleware/my_middleware.rb

class MyMiddleware
end
{% endhighlight %}

Also, another option is using the ```lib``` directory, but you have to make sure that 
the directory is in the autoload path. Or, you can manually require the middleware class
in the initializer.

As always, there are some conventions of how Rails middleware should be created. 
Like any other middleware class, the class needs an ```initialize``` method and a ```call``` method.
Also, the first argument of the initialize method is the application itself, and
the first argument of the call method is the request environment.

{% highlight ruby %}
# app/middleware/my_middleware.rb

class MyMiddleware
  def initialize app
    @app = app
  end

  def call env
    # do something...
  end
end
{% endhighlight %}

## Using our middleware

Rails allows us to mount this middleware class in the middleware stack so 
the application can use it in runtime. Again, you can see the middleware stack by running 
{% highlight bash %}
rake middleware
{% endhighlight %}
in your command line. 

### Mounting

Mounting middleware is usually done in the ```config/application.rb``` file.
But, Rails also allows us to mount different middleware for different environment. This means that
you can mount middleware in any of the ```config/environments``` files. Mounting a 
middleware class is done with the command:

{% highlight ruby %}
module MyRailsApplication
  class Application < Rails::Application
    *snip*

    config.middleware.use <class-name>, <first-argument>, <nth-argument>

    *snip*
  end
end
{% endhighlight %}

Or, in our case:

{% highlight ruby %}
# config/application.rb
module MyRailsApplication
  class Application < Rails::Application
    *snip*

    config.middleware.use "MyMiddleware"

    *snip*
  end
end
{% endhighlight %}

There is this tiny caveat when mounting middleware - note that the class name 
is written as a string, not as a constant. If you mount a middleware class in the
```config/application.rb``` file the class name has to be a string. But, if you load
the middleware class in the environment files (i.e. ```config/environment/development.rb```)
you can mount is as a constant:

{% highlight ruby %}
# config/environment/development.rb
module MyRailsApplication
  class Application < Rails::Application
    *snip*

    config.middleware.use MyMiddleware

    *snip*
  end
end
{% endhighlight %}

As far as I can tell, the reason for this tiny caveat is when Rails is loading 
all the required files on server boot, when ```config/application.rb``` is loaded 
not all constants are loaded. But, due to the nature of the envronment files, they 
seem to be autoloaded at the end, when all of the constants are present in memory.

### Mounting the middleware in the stack

Adding our middleware at a certain point of the middleware stack is done via 
the ```insert_before``` and ```insert_after``` commands. 

For example, if we, for whatever reason, want to add our ```MyMiddleware``` class 
just before the ```Rails::Rack::Logger``` middleware, we can use this line:

{% highlight ruby %}
module MyRailsApplication
  class Application < Rails::Application
    *snip*

    config.middleware.insert_before "Rails::Rack::Logger", "MyMiddleware"

    *snip*
  end
end
{% endhighlight %}

On the flipside, if we want to add our ```MyMiddleware``` class after the 
```Rails::Rack::Logger``` middleware, we can use this line:

{% highlight ruby %}
config.middleware.insert_after "Rails::Rack::Logger", "MyMiddleware"
{% endhighlight %}

Another convenience method that Rails provides us is ```swap```, as in swapping middleware. 
For example, if you wrote your own params parser middleware, that will substitute 
```ActionDispatch::ParamsParser```, you can swap it using:

{% highlight ruby %}
config.middleware.swap "ActionDispatch::ParamsParser", "MyParamsParser"
{% endhighlight %}

This will swap the middleware classes, as in, it will use ```MyParamsParser``` instead
of ```ActionDispatch::ParamsParser```. 

Just to give you an idea, this can be useful when debugging - you can extend the 
existing middleware, add some logging so you can see how the data mutates in the 
params parsers and continue with the normal execution of parsing the params.
One can enable that middleware just in development mode, so the middleware is swapped only in
development environment.

That being said, lets see how we can write our own tiny middleware and mount it to an existing Rails application.

## DeltaLogger

We will write a tiny middleware class that will calculate the delta time of the request
and log it to the Rails console. You can open any Rails application that you have
laying in your computer and play with it. I promise we won't do anything malicious. :-)

First, we need a middleware class. Lets add it to the ```app/middleware``` directory. 
It's worth mentioning that the directory does not exist by default. 
So, if it is missing - feel free to create it.

Next, we'll need to add the ```initialize``` and the ```call``` method. Remember, the first
argument of the ```initialize``` method is the application, and the first argument of the ```call``` method
is the request environment.

{% highlight ruby %}
# app/middleware/delta_logger.rb

class DeltaLogger
  def initialize app
    @app = app
  end

  def call env
    # do something...
  end
end
{% endhighlight %}

Next, we'll need to calculate the total time that the applicaiton took to process the request
and log it:

{% highlight ruby %}
# app/middleware/delta_logger.rb

class DeltaLogger
  def initialize app
    @app = app
  end

  def call env
    request_started_on = Time.now
    @status, @headers, @response = @app.call(env)
    request_ended_on = Time.now

    Rails.logger.debug "=" * 50
    Rails.logger.debug "Request delta time: #{request_ended_on - request_started_on} seconds."
    Rails.logger.debug "=" * 50

    [@status, @headers, @response]
  end
end

{% endhighlight %}

As you can see, this is quite trivial. We save the time before the request has been
passed onto the rest of the middleware stack and the time after the middleware has finished with
the request. Then, we subtract the start time from the end time and we get a number of seconds
that the request processing took.

What's cool in Rails middleware is that we have the Rails application available to us in 
the scope of the middleware. This allows us to use to Rails logger and log the delta time.

If you managed to add your middleware to the Rails app, go ahead and boot it. When
the server boots, issue any request to it. In the logs you will see something like:

{% highlight bash %}
==================================================
Request delta time: 1.40877 seconds.
==================================================
{% endhighlight %}

This is our ```DeltaLogger``` logging the delta time.

### Formatting the output 

In one of the examples in the introduction of this post, I mentioned the option
of passing arguments to the middleware class. Lets see how we can use this 
feature to improve the formatting of the ```DeltaLogger```.

Passing arguments is done by adding the arguments after the class name in the mounting command:

{% highlight ruby %}
config.middleware.use "MyMiddleware", "First Argument", { second: "argument" }, ["nth-argument"]
{% endhighlight %}

In our tiny example, we can send through the character that we want the output 
to be formatted with. Currently, our default formatting is done with the equals sign. 
Making this customizable is easily done by sending this character as the first argument:

{% highlight ruby %}
# app/middleware/delta_logger.rb

class DeltaLogger
  def initialize app, formatting_char = '='
    @app = app
    @formatting_char = formatting_char
  end

  def call env
    request_started_on = Time.now
    @status, @headers, @response = @app.call(env)
    request_ended_on = Time.now

    Rails.logger.debug @formatting_char * 50
    Rails.logger.debug "Request delta time: #{request_ended_on - request_started_on} seconds."
    Rails.logger.debug @formatting_char * 50

    [@status, @headers, @response]
  end
end

{% endhighlight %}

Now, we can change the output when adding the ```DeltaLogger``` to the middleware stack:

{% highlight ruby %}
config.middleware.use "DeltaLogger", "*"
{% endhighlight %}

It's worth mentioning that if your Rails application is running, when changing 
a middleware class you will have to reboot the application so the new changes in 
the middleware can be picked up. This happens because Rails loads the middleware only once - 
on boot.

Now, when we send a request in the logs we can see that the ```DeltaLogger``` output changed:

{% highlight bash %}
**************************************************
Request delta time: 1.40877 seconds.
**************************************************
{% endhighlight %}

### Logging levels

Another way to leverage arguments is to make the logging level customizable. For example, 
you might want to change the logging level. In Rails, there are six different 
logging levels: ```:debug```, ```:info```, ```:warn```, ```:error```, ```:fatal```, and ```:unknown```. 
Lets make the logging level customizable.

{% highlight ruby %}
# app/middleware/delta_logger.rb

VALID_LOG_LEVELS = [:debug, :info, :warn, :error, :fatal, :unknown]

class DeltaLogger
  def initialize app, log_level
    @app = app
    # Default to :info log level if the user sets an invalid log level.
    @log_level = VALID_LOG_LEVELS.include?(log_level) ?  log_level : :info 
  end

  def call env
    request_started_on = Time.now
    @status, @headers, @response = @app.call(env)
    request_ended_on = Time.now

    Rails.logger.send(@log_level, '=' * 50)
    Rails.logger.send(@log_level, "Request delta time: #{request_ended_on - request_started_on} seconds.")
    Rails.logger.send(@log_level, '=' * 50)

    [@status, @headers, @response]
  end
end

{% endhighlight %}

Now, in our application.rb (or environment.rb) file, we can set the logger to 
use the desired loging level:

{% highlight ruby %}
config.middleware.use "DeltaLogger", :warn
{% endhighlight %}

If we reboot the Rails app and send a new request, the logs will show:

{% highlight bash %}
[WARN] ==================================================
[WARN] Request delta time: 0.270595 seconds.
[WARN] ==================================================
{% endhighlight %}

Be aware that your output may vary because logging output relies on the [logger formatter](http://api.rubyonrails.org/classes/ActiveSupport/Logger/SimpleFormatter.html){:target="_blank"} that your application is using.

If you are not seeing the exact output, you can plug this logging formatter in your app:

{% highlight ruby %}
# lib/delta_formatter.rb

class DeltaFormatter < Logger::Formatter
  def call(severity, time, program_name, msg)
    "[#{severity}] #{String === msg ? msg : msg.inspect}\n"
  end
end
{% endhighlight %}

You can use this formatter by including it into ```application.rb```:

{% highlight ruby %}
*snipped*
require "./lib/delta_formatter"

module MyApplication
  class Application < Rails::Application
    config.autoload_paths += %W( #{config.root}/lib/**/*)

    *snipped*

    config.middleware.use "DeltaLogger", :warn
    config.log_formatter = DeltaFormatter.new
  end
end
{% endhighlight %}

After adding this, you'll need to reboot your server and you should see the output with
the severity tag.

## Thread safety

Last of the important topics about Rails middleware is thread-safety. When using Puma or Unicorn web servers,
one of the strong sides of these servers is that they are threaded. Since 
our middleware runs on these servers, it has to be thread-safe, meaning, it should 
easily spawn multiple duplicates of it so different threads can use different objects of
the same middleware. 

The easiest and most efficient way to do this is to ```dup``` the middleware object 
that is created in runtime. 

{% highlight ruby %}
# app/middleware/delta_logger.rb

VALID_LOG_LEVELS = [:debug, :info, :warn, :error, :fatal, :unknown]

class DeltaLogger
  def initialize app, log_level
    @app = app
    # Default to :info log level if the user sets an invalid log level.
    @log_level = VALID_LOG_LEVELS.include?(log_level) ? log_level : :info 
  end

  def call env
    dup._call env
  end

  def _call env
    request_started_on = Time.now
    @status, @headers, @response = @app.call(env)
    request_ended_on = Time.now

    Rails.logger.send(@log_level, '=' * 50)
    Rails.logger.send(@log_level, "Request delta time: #{request_ended_on - request_started_on} seconds.")
    Rails.logger.send(@log_level, '=' * 50)

    [@status, @headers, @response]
  end
end

{% endhighlight %}

By adding the ```_call``` method and duplicating the object we make sure that any instance variables 
we set in ```_call``` will be set on the duped instance, not the original. This allows the
web server to use a separate (duped) object for each thread which will contain different
data, based on it's execution.

## Outro

As you can see, there's quite a bit that needed to be covered in this post. I hope that
this explained how you can write your own Rails middleware and mount it in the middleware stack. 
It's very easy to get creative with Rails middleware as it iss highly customizable, quite 
low-level(ish) while you have the whole app available in runtime. Next time, we will work
together on building and testing a real-life example of Rails middleware. 

Until then, feel free to drop a comment, I would love to hear your opinions, problems and 
code that you've come up with.

