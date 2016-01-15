---
layout: post
title: "Migrating a test suite from RSpec to Minitest"
tags: [ruby, testing, tdd, rspec, minitest]
---

I have always wanted to have some fun with [Minitest](https://github.com/seattlerb/minitest){:target="blank"} but
until this weekend I never got the chance to do it. For those of you that don't know,
Minitest is a suite of testing facilities, that support TDD, BDD, mocking and benchmarking. 
Having wanted to play with Minitest, this weekend I decided that I will migrate the test 
suite of a gem of mine, from RSpec to Minitest. Read on to see how it all went.

The gem that I worked with is called [Forecastr](https://github.com/fteem/forecastr){:target="blank"}.
It is a very minimal gem that serves as a wrapper for the 
[Open Weather Map](http://openweathermap.org){:target="blank"} API. 
It supports only current forecast: temperature, pressure, humidity, min/max 
temperatures and wind (speed and direction).

## The setup

After checking out to a new Git branch (duh!) I first went to the gemspec and 
changed the RSpec dependency to Minitest.

{% highlight ruby %}
spec.add_development_dependency "minitest"
{% endhighlight %}

Setup-wise, the first difference that I noticed is that while RSpec's specs live in ```spec/forecastr```,
Minitest's tests live in ```test/forecastr```. Although this is the default, it's 
not the only way to do it. When creating the new Rake task for running the Minitest tests, which
I'll get to shortly, one can specify the path where she/he wants the tests to live.

In the newly created ```test/forecastr``` path, I had to add the ```test_helper.rb```. 
The purpose of the file is the same as ```spec_helper.rb``` in RSpec. Instead of requiring 
RSpec, now I had to require:

{% highlight ruby %}
require 'minitest/autorun'  # the test runner
require 'minitest/pride'    # adds some formatting to the test output
{% endhighlight %}

The last important thing was the Rake task. To run all the tests one needs to add
a built-in Rake task to the Rakefile:

{% highlight ruby %}
require "rake/testtask"

Rake::TestTask.new do |t|
  t.libs << 'test'
  t.pattern = "test/**/*_test.rb"
  t.warning = true
end

task default: :test
{% endhighlight %}

The ```Rake::TestTask``` object is the task that will be called when running ```rake test```
in command line. As you can see, it takes a small configuration block. In it, one can
set the path of the tests (what I mentioned before). On the last line, I made this 
task to be the default one, because it's more convenient for me and I don't need to 
run any other tasks.

## The first test

I started by migrating one test at a time from the spec directory to the test directory. 
What was interesting to me is that every test file in Minitest is a class that
inherits from ```Minitest::Test```. Also, every test case is a method, which of course
makes sense since the test file is a class. In contrast to RSpec, Minitest is very
verbose, while RSpec hides a lot of complexity for you with it's huge collection of
helper methods.

At the beginning I had a bit of problem with naming the methods because I was used
to the "free text" way of describing test cases in RSpec. I eventually got the first one:

{% highlight ruby %}
require 'test_helper'

class Forecastr::WindTest < Minitest::Test

  def setup
    @wind = Forecastr::Wind.new(2.5, -37)
  end

  def test_speed_in_ms
    assert_equal @wind.speed, "2.5 m/s"
  end

  def test_direction
    assert_equal @wind.direction, "NNW"
  end
end
{% endhighlight %}

What was interesting to me is that I got really cool warnings when running the tests:

{% highlight bash %}
/Users/ie/dev/forecastr/lib/forecastr/wind.rb:12: warning: method redefined; discarding old speed
/Users/ie/dev/forecastr/lib/forecastr/wind.rb:16: warning: method redefined; discarding old direction
{% endhighlight %}

The reason behind these errors is that, back in the day when I was writing the ```Wind``` class, 
I had added ```attr_reader``` for ```speed``` and ```direction```. Although I had these
in place, I had overridden the methods, so the warning was spot on. 

{% highlight ruby %}
module Forecastr
  class Wind
    DIRECTIONS = ["N","NNE","NE","ENE","E","ESE", "SE", "SSE","S","SSW","SW","WSW","W","WNW","NW","NNW"]

    attr_reader :speed, :direction # <--- useless...

    def initialize(speed, angle)
      @speed = speed
      @angle = angle
    end

    def speed
      "#{@speed} m/s"
    end

    def direction
      val = ((@angle/22.5) + 0.5).to_i
      DIRECTIONS[val % 16]
    end

    def to_s
      "#{speed} #{direction}"
    end
  end
end

{% endhighlight %}

After removing the unneeded attr_readers the warnings went away. 

{% highlight ruby %}
module Forecastr
  class Wind
    DIRECTIONS = ["N","NNE","NE","ENE","E","ESE", "SE", "SSE","S","SSW","SW","WSW","W","WNW","NW","NNW"]

    def initialize(speed, angle)
      @speed = speed
      @angle = angle
    end

    def speed
      "#{@speed} m/s"
    end

    def direction
      val = ((@angle/22.5) + 0.5).to_i
      DIRECTIONS[val % 16]
    end

    def to_s
      "#{speed} #{direction}"
    end
  end
end

{% endhighlight %}

Although this is a nice feature of Minitest, I ended up turning it off because it started
reporting warnings for libraries that weren't under my control:

{% highlight bash %}
/Users/ie/.rbenv/versions/2.2.2/lib/ruby/gems/2.2.0/gems/webmock-1.15.0/lib/webmock/http_lib_adapters/net_http.rb:100: warning: assigned but unused variable - response
{% endhighlight %}

After some fun with the test, I got it passing:

![http://i.imgur.com/Z8fIdPI.png](http://i.imgur.com/Z8fIdPI.png)

Fabulous!

## The flow

After migrating two classes from RSpec to Minitest, I noticed a change in my workflow.
I was running ```rake```, which runs the complete suite, to run just one test. 
The only way I could find to run a single test seems a bit too verbose for me.
Since I am very used to RSpec, running a test with the line number is one of
my most used features of RSpec. 

In this sense, Minitest gave me a tiny disappointment. Running a huge command
with four arguments every time I want to run a single test is a flow-killer. True, I could 
use Guard to run the test when it gets changed. But also, I often want to run 
tests by hand without any hassle. 

Thanks to Nick Quaranto who wrote the [m](https://github.com/qrush/m){:target="blank"} gem. 
The gem is quite simple - just a Test::Unit runner that can run tests by line number.
Although simple, it is exactly what I needed!

This means that, instead of running a single test with:

{% highlight bash %}
ruby -Itest test/lib/test.rb --name /some_test/
{% endhighlight %}

**m** allowes one to do the same with:

{% highlight bash %}
m test/lib/test.rb:12
{% endhighlight %}

Fabulous!

## Minitest::Spec::DSL

After getting all my tests green, I took a look at Minitest's Spec DSL. Mintest::Spec is
a functionally complete spec engine. It turns the test assertions 
into spec expectations. This basically allows the developers to use Minitest just as RSpec. 
The syntax is pretty much the same, so migrating RSpec suite to Minitest is much much
easier with Minitest::Spec in the mix. 

My personal preference in this case was to stay away from spec expectations and 
think more in a test assertions way. Although they are basically the same, this 
experience was much more joyful for me. Or maybe I just needed to have fun with something
new.

## Output Formatting

The basic output, or the one that comes with ```minitest/pride``` wasn't really cutting it for me.
When looking for other formatting options, I ran into [minitest-reporters](https://github.com/kern/minitest-reporters){:target="blank"} -
a gem for creating customizable Minitest output formats.

The setup is quite easy - just require the library in the ```test_helper.rb``` and 
call ```Minitest::Reporters.use!```:

{% highlight ruby %}
require "forecastr"
require "minitest/autorun"
require "minitest/reporters"

Minitest::Reporters.use! Minitest::Reporters::SpecReporter.new
{% endhighlight %}

The gem has couple of built-in reporters, and I went for the ```Minitest::Reporters::SpecReporter```.
With this gem one can also create his own reporters and formatting.

## So, what's next?

Although Forecastr's codebase might not be big with lots of challenges in it, this was defnitely
a really nice exercise for a hot Saturday afternoon. I just wish I had the chance to work
with Fixtures and some fake objects. 

All in all, what I can say now is that using Minitest was a real joy. It seems to be 
very simple to setup, extend and use. But, although simple, in my opinion it is 
as powerful as the other alternatives. 

You can see the whole RSpec to Minitest migration on [this commit](https://github.com/fteem/forecastr/commit/39264e419dc932f4562d622a293d920634218af6){:target="blank"}.
