---
layout: post
title: "How to set tests as pending in your ExUnit suite"
tags: [elixir, exunit, testing]
---

Elixir's built in testing library is called ExUnit. It's a proper testing framework, 
which, although simple, gives the developers a lot of power and flexibility. If 
you come from Ruby land, I am sure you've been in a position where you want to 
set a certain test to be skipped. For example, RSpec in Ruby does it with the 
`pending` method. Let's see how we can customize our test suite so ExUnit 
can skip over tests in our test suite. 

## Tags

ExUnit comes with this neat feature called **tags**. Tags are basically module variables
in the test module. In the context of the test, you can think of them as **annotations**.
So, by tagging a test, the value of the tag can be used in the test itself, by passing
the context. 

{% highlight elixir %}
defmodule FileTest do
  # Changing directory cannot be async
  use ExUnit.Case, async: false

  setup context do
    # Read the :cd tag value
    if cd = context[:cd] do
      prev_cd = File.cwd!
      File.cd!(cd)
      on_exit fn -> File.cd!(prev_cd) end
    end

    :ok
  end

  @tag cd: "fixtures"
  test "reads utf-8 fixtures" do
    File.read("hello")
  end
end
{% endhighlight %}

<small>Note: Example taken from ExUnit's official 
[documentation](http://elixir-lang.org/docs/v1.0/ex_unit/ExUnit.Case.html).</small>

Now, how can we utilize tags to get RSpec's `pending` funcionality in ExUnit?

## Running tests by tag

Running tests by tag is really simple in ExUnit. Basically, ExUnit has the ability 
to skip/include any test by it's tag.

For example, let's say you have tagged couple of tests that are brittle:

{% highlight elixir %}
defmodule SomeTest do
  use ExUnit.Case, async: false

  @tag :brittle
  test "some brittle code" do
    # test here...
  end
end
{% endhighlight %}

Now, you can run only the tests with the `brittle` tag, by executing:

{% highlight bash %}
mix test test/some_test.exs --include brittle
{% endhighlight %}

Having this in mind, let's introduce a custom tag of ours so we can make our tests 
`pending`.

## Custom tags

Although the word "custom" used in programming often means something complicated 
done from scracth by the programmer, in this case it's very simple. 

Let's take any test and tag it with `:pending`.

{% highlight elixir %}
defmodule SomeTest do
  use ExUnit.Case, async: false

  @tag :pending
  test "always true" do
    assert 1 == 1
  end
end
{% endhighlight %}

We can exclude this "pending" test by executing our test suite with:

{% highlight bash %}
mix test --exclude pending
{% endhighlight %}

This will skip any tests that have the `:pending` tag. Simple as that, we have
RSpec's `pending` functionality just by excluding a tag. If you are wondering
if we can make the exclusion of the tag automatic - yes we can. In your
`test_helper.exs` add this line, before the `ExUnit.start` line:

{% highlight elixir %}
ExUnit.configure(exclude: [pending: true])
{% endhighlight %}

This configuration will tell ExUnit that it should always skip tests that are tagged as
`:pending`. Now, every time you run your tests, the tests tagged with `:pending`
will be excluded.

Do you have any other interesting configuration trick in your ExUnit suite? Feel free
to share them in the comments, I would love to learn more about ExUnit!

Thanks for reading!
