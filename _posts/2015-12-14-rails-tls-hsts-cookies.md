---
layout: post
title: "Rails, Secure Cookies, HSTS and friends"
excerpt: "<p>Learn more about how secure cookies and HTTP Strict Transport 
Security work in Rails by taking a look in it's middleware stack</p>"
tags: [ruby, rails, hsts, cookies, tls, ssl]
---

Ruby on Rails as a framework does a lot of things for us developers. We get a 
very customizable middleware stack, great routing system, very expressive ORM, 
helpful modules with great utility methods in them and so on. But in Rails 
there's more than meets the eye. It does some great things that we just take for 
granted or on occasions we don't even know they exist. 

Some of these features are TLS redirection, secure cookies and HTTP Strict 
Transport Security (HSTS). Let's dive in into the Rails middleware stack and see 
what these things mean and what benefits they provide. 

## HTTP Strict Transport Security

According to [Wikipedia](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security){:target="_blank"}:

> HTTP Strict Transport Security (HSTS) is a web security policy mechanism which 
> helps to protect secure HTTPS websites against downgrade attacks and cookie 
> hijacking. It allows web servers to declare that web browsers (or other 
> complying user agents) should only interact with it using secure HTTPS 
> connections and never via the insecure HTTP protocol. HSTS is an IETF 
> standards track protocol and is specified in 
> [RFC 6797](https://tools.ietf.org/html/rfc6797){:target="_blank"}.
>
> The HSTS Policy is communicated by the server to the user agent via an HTTP 
> response header field named "Strict-Transport-Security". HSTS Policy specifies 
> a period of time during which the user agent shall access the server in a 
> secure-only fashion.

This is a short and nice summary of HSTS. The Internet Engineering Task Force 
(IETF) have solved the problem by making web servers send a HTTP response header
which will make the browsers use HTTPS over HTTP when requesting any of the 
resources on that particular web server.

If you take a deeper look, into the 
[Request for Comments (RFC) No. 6797](https://tools.ietf.org/html/rfc6797) where
HSTS was proposed, you will see this: 

> **2.3.  Threat Model**
>
>  HSTS is concerned with three threat classes: passive network
>  attackers, active network attackers, and **imperfect web developers**.
>  However, it is explicitly not a remedy for two other classes of
>  threats: phishing and malware.  Threats that are addressed, as well
>  as threats that are not addressed, are briefly discussed below.

It's a good thing that the IETF and the Rails core developers know that we are 
*imperfect* (read: lazy), so they have our backs. \*wink emoji\*

Since we got the basics right, let's look at the structure of the HSTS header:

`Strict-Transport-Security: max-age=31536000; includeSubdomains; preload`

It is composed of three directives: `max-age`, `includeSubdomains` and `preload`.
The `max-age` directive tells the browser that this is the duration of time for 
which HSTS will be active for the domain. The `includeSubdomains` directive is 
quite self-explanatory: it tells the browser that HSTS will be active for all 
subdomains. The last one, `preload`, is created by the Chrome security team. 
It's purpose is to create a list of domains that will be **preloaded** to 
Chrome, so Chrome knows that HSTS will be preloaded for the given domain. Later, 
this preloading mechanism got incorporated to Firefox, Safari and Internet 
Explorer. This allows HSTS to kick in even for the first visit of the website.

## Secure cookies

Cookies, the ones that browsers consume, have multiple values and flags on them. 
Here's a screenshot of someone's cookies as shown in the Chrome Developer Tools:

![Cookies](http://i.imgur.com/7GEcADR.jpg)

As you can see in the screenshot, a cookie has a name, a value, the domain (or 
owner), the path, expiry date/time, it's size, the HttpOnly flag, the **Secure** 
flag and the First-Party field. For the purpose of this article, we are only 
interested in the Secure flag.

The secure flag tells the browser that it can send this cookie to the owner 
**only via HTTPS**. This protects the user when under a Man in the middle (MITM) 
attack. Basically, if someone steals the session cookie from the user, it will 
always be encrypted with SSL/TLS so it will be unusable for the attacker.

## SSL/TLS Redirection

In comparison to HSTS and Secure Cookies this is a really simple security 
mechanism. Rails has the ability to redirect clients accessing it from HTTP to 
HTTPS. Think of it in this way - if the proper configuration is set, it will 
check if the request comes via HTTPS. If not, it redirects the client to the 
same URL, just via HTTPS. Simple as that.

## Back to Rails

Now, how does Rails implement these mechanisms? Think about this: when we are
fetching/building the data for the response, whether it's XML, JSON or HTML, we
rarely do anything with the response headers. We usually render some 
document/data and we let Rails take care of the rest. So, there has to be some 
configuration where we can turn on TLS redirection, secure cookies and HSTS.

Rails configuration can be found in multiple places. If it's configuration 
per environment, it's usually `config/<environment>.rb`. If it's general 
configuration - `config/application.rb`. Other times, it can be in 
`config/initializers`. It very much depends on what part of the application you
want to configure. 

By default Rails has some configurations set up for us. For example, when a 
brand new Rails application is generated, in the `config/production.rb` file you 
can see the following lines:

{% highlight ruby %}
# Force all access to the app over SSL, use Strict-Transport-Security, 
# and use secure cookies.
config.force_ssl = true
{% endhighlight %}

As you can notice in the comment, this is the line that enables all these 
security mechanisms. Quite self-descriptive, the attribute is called `force_ssl`. 
For production environment, this is set to `true` by default. Let's see how this 
actually works under the hood.

### Where does it do it? 

Searching through the Rails source code, starting from the 
[`Rails::Application::Configuration`](https://github.com/rails/rails/blob/b11bca98bf5431ce9522c6b5707f51cd8d7f401c/railties/lib/rails/application/configuration.rb#L35){:target="_blank"} 
where I noticed that the `force_ssl` configuration is set to `false` by default. 
But, this was a dead end - it was only the Rails configuration object, nothing 
else. 

Searching on for the `force_ssl` property, I ran into 
`Rails::Application::DefaultMiddlewareStack` 
([source](https://github.com/rails/rails/blob/b11bca98bf5431ce9522c6b5707f51cd8d7f401c/railties/lib/rails/application/default_middleware_stack.rb#L14)). 
In the `#build_stack` method I noticed that when Rails is booting up, it decides 
if the `Rails::Application::ActionDispatch::SSL` middleware should be pushed onto 
the middleware stack. This was the obvious place to look, so let's open that 
class.

## Enforcing TLS/SSL and HSTS

In the `Rails::Application::ActionDispatch::SSL` there's the `build_hsts_header` 
method:

{% highlight ruby %}
def build_hsts_header(hsts)
  value = "max-age=#{hsts[:expires].to_i}"
  value << "; includeSubDomains" if hsts[:subdomains]
  value << "; preload" if hsts[:preload]
  value
end
{% endhighlight %}

It is being called from the `#initialize` method, which is where an object from
the middleware class is created:

{% highlight ruby %}
def initialize(app, redirect: {}, hsts: {}, **options)
  @app = app

  if options[:host] || options[:port]
    ActiveSupport::Deprecation.warn <<-end_warning.strip_heredoc
    The `:host` and `:port` options are moving within `:redirect`:
    `config.ssl_options = { redirect: { host: …, port: … }}`.
    end_warning
    @redirect = options.slice(:host, :port)
  else
    @redirect = redirect
  end

  @hsts_header = build_hsts_header(normalize_hsts_options(hsts))
end
{% endhighlight %}

The `#build_hsts_header` method takes a normalized hash of HSTS options and 
builds the header based on the values in the hash. This all takes effect in the
`#call` method:

{% highlight ruby %}
def call(env)
  request = Request.new env

  if request.ssl?
    @app.call(env).tap do |status, headers, body|
      set_hsts_header! headers
      flag_cookies_as_secure! headers
    end
  else
    redirect_to_https request
  end
end
{% endhighlight %}

After the HSTS header is added to the request, it is passed down onto the rest
of the middleware stack. 

## Securing the cookies

Securing the cookies is done in the same module. The `flag_cookies_as_secure!` 
method looks for the `Set-Cookie` header in the request and appends the `secure` 
property to the `Set-Cookie` header if needed. 

{% highlight ruby %}
def flag_cookies_as_secure!(headers)
  if cookies = headers['Set-Cookie'.freeze]
    cookies = cookies.split("\n".freeze)

    headers['Set-Cookie'.freeze] = cookies.map { |cookie|
      if cookie !~ /;\s*secure\s*(;|$)/i
        "#{cookie}; secure"
      else
        cookie
      end
    }.join("\n".freeze)
  end
end
{% endhighlight %}

For reference, this is what the `Set-Cookie` header looks like:

{% highlight bash %}
Set-Cookie: cookie-name=cookie-value-here; path=/; expires=Fri, 01 Jan 2016 00:00:00 -0000; secure; HttpOnly
{% endhighlight %}

This will tell the browser that the cookie should be secured and sent only via 
HTTPS. Just like the HSTS header, the secure cookies header is attached to the
request in the `#call` method (which you can see above). 

Note: do not confuse the `secure` directive with the `HttpOnly` directive.

## TLS redirection

Another feature of Rails is the so-called "TLS redirection". It's workings are 
quite simple - whenever you have `force_ssl` set to true, it will redirect all of
the HTTP traffic to HTTPS. Again, the `Rails::Application::ActionDispatch::SSL`
middleware class is responsible for this behaviour of Rails, more specifically 
the `redirect_to_https` method, which invokes the `https_location_for` method:

{% highlight ruby %}
def redirect_to_https(request)
  [ @redirect.fetch(:status, 301),
    { 'Content-Type' => 'text/html',
      'Location' => https_location_for(request) },
    @redirect.fetch(:body, []) ]
end

def https_location_for(request)
  host = @redirect[:host] || request.host
  port = @redirect[:port] || request.port

  location = "https://#{host}"
  location << ":#{port}" if port != 80 && port != 443
  location << request.fullpath
  location
end
{% endhighlight %}

As you can notice, `redirect_to_https` rebuilds a response which will have the
same `Content-Type` and `body`, but the `Location` and `HTTP Status` will be 
different. 
In it's core, the middleware just wraps the incoming request and checks if it's
SSL/TLS based. If not, it redirects the web client to the same resource on the 
server, just via HTTPS. 

## Outro

As we saw in this article, Rails does some magic under the hood for us. It's 
really great that we all get these great benefits without even bothering to set
them up. Although we get them by default, it's good to know what happens with
our application under the hood. 

I hope this article was fun to read and informative for you. Do you have a 
favourite "hidden" functionality of Rails that not many of us know? Please share
your thoughts with me in the comments section below.

