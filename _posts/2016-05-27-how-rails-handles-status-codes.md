---
layout: post
title: "How Rails handles status codes"
tags: [ruby, decorators, presenters, odd]
image: how-rails-handles-status-codes.png
---

Recently, I have been building an API as part of my day job. Rails is a great
framework to build APIs in, and it has been a joy so far. When building the
responses of the API, it's paramount to understand what HTTP statuses you should
utilise, which will in return help the consumers by providing more meaningful 
responses. 

Sure, you could always have a `status` property in the response JSON, which will 
be a human-readable status code. But, I would like you to think about HTTP status 
codes as a nice card on a present, with your beautiful handwriting and best 
wishes to whom the gift goes to. HTTP status code don't just add more semantical 
correctness, but they also speak about the nature of the response.

## Rails and statuses

Ruby on Rails, being a great framework, adds a very nice layer of abstraction of
the status codes, and allows the developers to easily use custom status codes in
their responses. I am sure, at some point you have seen something like:

{% highlight ruby %}
render json: @resource, status: 201 # Created
{% endhighlight %}

While this might be sometimes redundant, the `render` method in Rails allows you
to specify HTTP status codes for the response. But also, being very developer
friendly, Rails allows you to do the same with "human readable" status codes:

{% highlight ruby %}
render json: @resource, status: :created
{% endhighlight %}

The two code samples are identical, they specify the HTTP 201 status code to the
response. But, what are the available Rails status codes (as symbols) that you
can use? And how does Rails actually do this?

## Status codes

The HTTP status codes list is a quite stable and static list. Although sometimes
status codes are added to the list, once you learn it, keeping up to date is
rather easy. The latest addition to this list is HTTP 451 - Unavailable For 
Legal Reasons. You can see the RFC where it was proposed 
[here](http://tools.ietf.org/html/rfc7725). 

So, how does Rails knows how to create all of these symbols, so we can use them
in our `render` methods? Well, what's very interesting is that Rails actually
doesn't do much about this. It relies on Rack, which does all of this magic. In
an `irb` session, type:

{% highlight ruby %}
>> Rack::Utils::HTTP_STATUS_CODES

{100=>"Continue",
 101=>"Switching Protocols",
 102=>"Processing",
 200=>"OK",
 201=>"Created",
 202=>"Accepted",
 203=>"Non-Authoritative Information",
 204=>"No Content",
 205=>"Reset Content",
 206=>"Partial Content",
 207=>"Multi-Status",
 208=>"Already Reported",
 226=>"IM Used",
 300=>"Multiple Choices",
 301=>"Moved Permanently",
 302=>"Found",
 303=>"See Other",
 304=>"Not Modified",
 305=>"Use Proxy",
 307=>"Temporary Redirect",
 308=>"Permanent Redirect",
 400=>"Bad Request",
 401=>"Unauthorized",
 402=>"Payment Required",
 403=>"Forbidden",
 404=>"Not Found",
 405=>"Method Not Allowed",
 406=>"Not Acceptable",
 407=>"Proxy Authentication Required",
 408=>"Request Timeout",
 409=>"Conflict",
 410=>"Gone",
 411=>"Length Required",
 412=>"Precondition Failed",
 413=>"Payload Too Large",
 414=>"URI Too Long",
 415=>"Unsupported Media Type",
 416=>"Range Not Satisfiable",
 417=>"Expectation Failed",
 422=>"Unprocessable Entity",
 423=>"Locked",
 424=>"Failed Dependency",
 426=>"Upgrade Required",
 428=>"Precondition Required",
 429=>"Too Many Requests",
 431=>"Request Header Fields Too Large",
 500=>"Internal Server Error",
 501=>"Not Implemented",
 502=>"Bad Gateway",
 503=>"Service Unavailable",
 504=>"Gateway Timeout",
 505=>"HTTP Version Not Supported",
 506=>"Variant Also Negotiates",
 507=>"Insufficient Storage",
 508=>"Loop Detected",
 510=>"Not Extended",
 511=>"Network Authentication Required"}
{% endhighlight %}

As you can see, the `Rack::Utils::HTTP_STATUS_CODES` is a Hash that has all of
the HTTP status codes, with the corresponding messages. If you open the 
`Rack::Utils` 
[documentation](http://www.rubydoc.info/github/rack/rack/master/Rack/Utils#HTTP_STATUS_CODES-constant)
you can see how these are programmatically created, including the code that
does the conversion. The status codes are pulled from 
[this CSV file](www.iana.org/assignments/http-status-codes/http-status-codes-1.csv)
and are parsed using this piece of code:

{% highlight ruby %}
ruby -ne 'm = /^(\d{3}),(?!Unassigned|\(Unused\))([^,]+)/.match($_) and puts "#{m[1]} => \x27#{m[2].strip}\x27,"'
{% endhighlight %}

Great. But, how do we get the symboled versions of the status messages? Well,
Rack does that for us as well. In an `irb` session, try this:

{% highlight ruby %}
>> Rack::Utils::SYMBOL_TO_STATUS_CODE
{:continue=>100,
 :switching_protocols=>101,
 :processing=>102,
 :ok=>200,
 :created=>201,
 :accepted=>202,
 :non_authoritative_information=>203,
 :no_content=>204,
 :reset_content=>205,
 :partial_content=>206,
 :multi_status=>207,
 :already_reported=>208,
 :im_used=>226,
 :multiple_choices=>300,
 :moved_permanently=>301,
 :found=>302,
 :see_other=>303,
 :not_modified=>304,
 :use_proxy=>305,
 :temporary_redirect=>307,
 :permanent_redirect=>308,
 :bad_request=>400,
 :unauthorized=>401,
 :payment_required=>402,
 :forbidden=>403,
 :not_found=>404,
 :method_not_allowed=>405,
 :not_acceptable=>406,
 :proxy_authentication_required=>407,
 :request_timeout=>408,
 :conflict=>409,
 :gone=>410,
 :length_required=>411,
 :precondition_failed=>412,
 :payload_too_large=>413,
 :uri_too_long=>414,
 :unsupported_media_type=>415,
 :range_not_satisfiable=>416,
 :expectation_failed=>417,
 :unprocessable_entity=>422,
 :locked=>423,
 :failed_dependency=>424,
 :upgrade_required=>426,
 :precondition_required=>428,
 :too_many_requests=>429,
 :request_header_fields_too_large=>431,
 :internal_server_error=>500,
 :not_implemented=>501,
 :bad_gateway=>502,
 :service_unavailable=>503,
 :gateway_timeout=>504,
 :http_version_not_supported=>505,
 :variant_also_negotiates=>506,
 :insufficient_storage=>507,
 :loop_detected=>508,
 :not_extended=>510,
 :network_authentication_required=>511}
{% endhighlight %}

As you can see, the `Rack::Utils::SYMBOL_TO_STATUS_CODE` constant contains all
of the status codes as symbols. The 
[documentation](http://www.rubydoc.info/github/rack/rack/master/Rack/Utils#SYMBOL_TO_STATUS_CODE-constant)
also shows the actual conversion, from `HTTP_STATUS_CODES` to 
`SYMBOL_TO_STATUS_CODE`:

{% highlight ruby %}
Hash[*HTTP_STATUS_CODES.map { |code, message|
  [message.downcase.gsub(/\s|-|'/, '_').to_sym, code]
  }.flatten]
{% endhighlight %}

Having all of this documentation in place, is rather easy to see how everything
falls into place. Rails being a Rack app, can utilise everything that Rack 
provides, an this is just an example. But, how does actually Rails plugs this
into it's runtime and has the ability to understand what is the status code?

## A bit deeper

Rails' source code, although very documented and well structured, can still be
overwhelming to someone that hasn't spent enough time digging through. If you are
a Ruby developer, I encourage you to spend some time around Rails' code - you
will learn a lot, I promise you!

So, how does Rails know what status code to apply to the response, if you provide
just the symbolized version on the status? Well, let's start with Rails' class
hirearchy, which is quite nice. 

On the surface, or what we usually see as developers, the rendering is done in 
the controllers. If you open any of your applications' `ApplicationController`,
you will see something like:

{% highlight ruby %}
class ApplicationController < ActionController::Base
  ...
end
{% endhighlight %}

This means that all of our controllers are subclasses of the 
`ActionController::Base` class. If you open the 
[source code](https://github.com/rails/rails/blob/52ce6ece8c8f74064bb64e0a0b1ddd83092718e1/actionpack/lib/action_controller/base.rb) 
of the `ActionController::Base` class, you will see some very interesting things, 
that can be quite informative. But for our use case, we are interested in the 
following:

{% highlight ruby %}
MODULES = [
  AbstractController::Rendering,
  AbstractController::Translation,
  AbstractController::AssetPaths,

  Helpers,
  UrlFor,
  Redirecting,
  ActionView::Layouts,
  Rendering,
  Renderers::All,
  ConditionalGet,
  EtagWithTemplateDigest,
  Caching,
  MimeResponds,
  ImplicitRender,
  StrongParameters,

  Cookies,
  Flash,
  FormBuilder,
  RequestForgeryProtection,
  ForceSSL,
  Streaming,
  DataStreaming,
  HttpAuthentication::Basic::ControllerMethods,
  HttpAuthentication::Digest::ControllerMethods,
  HttpAuthentication::Token::ControllerMethods,

  # Before callbacks should also be executed the earliest as possible, so
  # also include them at the bottom.
  AbstractController::Callbacks,

  # Append rescue at the bottom to wrap as much as possible.
  Rescue,

  # Add instrumentations hooks at the bottom, to ensure they instrument
  # all the methods properly.
  Instrumentation,

  # Params wrapper should come before instrumentation so they are
  # properly showed in logs
  ParamsWrapper
]

MODULES.each do |mod|
  include mod
end
{% endhighlight %}

This piece of code, actually includes all of these modules in the
`ActionController::Base` class, which in return gives us all of the 
functionlities that our controllers have. Just look at the module names, like
`Redirecting`, `ActionView::Layouts`, `Rendering`, `Cookies`, `Flash` and so on.
Very self-explanatory, right?

Well, the classes that we are interested in are `AbstractController::Rendering`
and `ActionController::Rendering`. The first one is an abstract class, kind of
like an interface, and the second one is the default implementation class, which
contains all of the rendering mechanisms that Rails uses to render your response. 
When we call `render` in our controllers, we are executing the 
`ActionController::Rendering#render` method, which does couple of things for us: 
it normalizes the arguments, the options and then builds the body of the response. 
When it normalizes the arguments, deep inside the `_normalize_render` method, it 
contains these lines of code:

{% highlight ruby %}
if options[:status]
  options[:status] = Rack::Utils.status_code(options[:status])
end
{% endhighlight %}

This piece of code, takes the `:status` option from our render call, and
translates the symbol to an actual HTTP code. If we go back to `Rack::Utils`
documentation, we can see the actual implementation of the `status_code` method:

{% highlight ruby %}
# File 'lib/rack/utils.rb', line 573

def status_code(status)
  if status.is_a?(Symbol)
    SYMBOL_TO_STATUS_CODE[status] || 500
  else
    status.to_i
  end
end
{% endhighlight %}

Here's how your `status: :created` gets translated to a `status: 201`. As you
can see, although Rails' source is sometimes hard to digest, it's very well
done and navigating through it is not that hard. 

## What about exceptions?

So now we know how Rails does the expected rendering, which was created by us,
developers. But, what happens when Rails hits an exception of some sort, but it
still has to recover from it and return a proper HTTP status code to the client?

Although at first you might expect some sort of a `rescue` block in 
`ActionController::Metal` class, this is not the case. Open any Rails app you
have on your computer, and run in it's root directory:

{% highlight bash %}
➜  rake middleware

use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::LoadInterlock
use ActiveSupport::Cache::Strategy::LocalCache::Middleware
use Rack::Runtime
use ActionDispatch::RequestId
use Rails::Rack::Logger
*use ActionDispatch::ShowExceptions*
use ActionDispatch::DebugExceptions
use ActionDispatch::RemoteIp
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::Migration::CheckPending
use ActiveRecord::ConnectionAdapters::ConnectionManagement
use ActiveRecord::QueryCache
use Rack::Head
use Rack::ConditionalGet
use Rack::ETag
use ActionView::Digestor::PerRequestDigestCacheExpiry
run YourApp::Application.routes
{% endhighlight %}

Actually, the exceptions are handled in Rails' middleware, more specifically in 
the `ActionDispatch::ShowExceptions` middleware class. It wraps a group of 
exceptions and maps them to a specific status code. This is the code that does 
the mapping:

{% highlight ruby %}
module ActionDispatch
  class ExceptionWrapper
    cattr_accessor :rescue_responses
    @@rescue_responses = Hash.new(:internal_server_error)
    @@rescue_responses.merge!(
      'ActionController::RoutingError'              => :not_found,
      'AbstractController::ActionNotFound'          => :not_found,
      'ActionController::MethodNotAllowed'          => :method_not_allowed,
      'ActionController::UnknownHttpMethod'         => :method_not_allowed,
      'ActionController::NotImplemented'            => :not_implemented,
      'ActionController::UnknownFormat'             => :not_acceptable,
      'ActionController::InvalidAuthenticityToken'  => :unprocessable_entity,
      'ActionController::InvalidCrossOriginRequest' => :unprocessable_entity,
      'ActionDispatch::ParamsParser::ParseError'    => :bad_request,
      'ActionController::BadRequest'                => :bad_request,
      'ActionController::ParameterMissing'          => :bad_request,
      'Rack::Utils::ParameterTypeError'             => :bad_request,
      'Rack::Utils::InvalidParameterError'          => :bad_request
    )
    ...
  end
end
{% endhighlight %}

All of the exceptions that can occur in the request/response lifecycle are mapped 
to a specific status code, which will be returned by the middleware if an
exception is rescued. You can see this functionality in 
`ActionDispatch::ShowExceptions#call` method, which invokes the 
`render_exception` method if an `Exception` is rescued:

{% highlight ruby %}
def call(env)
  request = ActionDispatch::Request.new env
  @app.call(env)
rescue Exception => exception
  if request.show_exceptions?
    render_exception(request, exception)
  else
    raise exception
  end
end
{% endhighlight %}

{% highlight ruby %}
def render_exception(request, exception)
  backtrace_cleaner = request.get_header 'action_dispatch.backtrace_cleaner'
  wrapper = ExceptionWrapper.new(backtrace_cleaner, exception)
  status  = wrapper.status_code
  request.set_header "action_dispatch.exception", wrapper.exception
  request.set_header "action_dispatch.original_path", request.path_info
  request.path_info = "/#{status}"
  response = @exceptions_app.call(request.env)
  response[1]['X-Cascade'] == 'pass' ? pass_response(status) : response
rescue Exception => failsafe_error
  $stderr.puts "Error during failsafe response: #{failsafe_error}\n  #{failsafe_error.backtrace * "\n  "}"
  FAILSAFE_RESPONSE
end
{% endhighlight %}

Now, when the exception is being rendered, the `ExceptionWrapper` comes into the
picutre. It will wrap the exception, and return an approprite status code, which
is translated from the symbolized variant, to the number version, using:

{% highlight ruby %}
def status_code
  self.class.status_code_for_exception(@exception.class.name)
end
{% endhighlight %}

The `status_code` method invokes the `status_code_for_exception` method, which
converts the status code:

{% highlight ruby %}
def self.status_code_for_exception(class_name)
  Rack::Utils.status_code(@@rescue_responses[class_name])
end
{% endhighlight %}

And that's about it. After that, the middleware stack is being executed in order
to return the new response, with the exception and the proper HTTP status code 
attached.

## After all

Although all of this Rails "magic" might be a bit much to digest at first, 
Rails' source code does quite a good job of explining what is going on. And as
you can see, there's always something more than meets the eye. If you need to
remember something from this blogpost is that Rails does a very good 
(and interesting) job at handling HTTP status codes, to make your app's responses
semantically correct and client friendly. 

If you got to this point - thanks so much for reading. I know it was a bit of a
long journey, and I hope it was infromative for you. Looking forward to reading
your comments!


