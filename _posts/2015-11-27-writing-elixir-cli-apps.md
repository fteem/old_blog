---
layout: post
title: "Writing command line apps with Elixir"
tags: [elixir, cli, apps, escript]
---

Elixir is a really cool language. Although I do not have much experience with it 
(yet), I am always trying to build interesting stuff with it and learn the 
built-in tools. In this blog post I decided to show you how to build a 
self-contained command line application with Elixir, with some help from 
`escript`.

## Escript

Erlang and Elixir have this cool thing called escript. It's basically a tool 
that compiles an Elixir app that you have as a command line application. 

From Elixir's [documentation on escript](http://elixir-lang.org/docs/master/mix/Mix.Tasks.Escript.Build.html):

> An escript is an executable that can be invoked from the command line. An 
> escript can run on any machine that has Erlang installed and by default does 
> not require Elixir to be installed, as Elixir is embedded as part of the 
> escript.

> This task guarantees the project and its dependencies are compiled and packages 
> them inside an escript.

What is really interesting is that escript will build your Elixir CLI 
application and create an executable. The executable will only require Erlang to 
be able to run on any machine. It will not need Elixir because it will be 
contained in the executable it builds.

## `eight_ball` wrapped in `escript`

Now, instead of going at length of building a tiny example Elixir application, 
I will use the `eight_ball` application that we built in the 
[Write and publish your first Elixir library](/writing-elixir-library/) post. If 
you haven't read it yet you can click on the link above, read the step-by-step 
tutorial and come back to this post once you are done. 

The application is quite simple. It's a module which has only one function 
`EightBall.ask/1`. It takes a question as an argument and returns the answer. 
Let's use `escript` to build a command line application from this Elixir 
application. 

## Adding escript to the application

If you want to follow along, you can find the repo 
[here](https://github.com/fteem/eight_ball).

In the `mix.exs` file, we need to make an addition:

{% highlight elixir %}
defmodule EightBall.Mixfile do
  use Mix.Project

  def project do
    [app: :eight_ball,
     version: "0.0.1",
     elixir: "~> 1.0",
     build_embedded: Mix.env == :prod,
     start_permanent: Mix.env == :prod,
     escript: [main_module: EightBall.CLI], # <- this line
     deps: deps,
     package: package ]
  end
  # ...
end
{% endhighlight %}

The line we added will tell `escript` that the main module where the `main/1` 
function will be placed is `EightBall.CLI`. If you are wondering what `main/1` 
function, think of it in this way: the `main/1` function is the function that 
will be an entry point for the command line application. 

## `EightBall.CLI`

Since the `EightBall.CLI` module doesn't exist, let's create it:

{% highlight elixir %}
defmodule EightBall.CLI do
  def main(args) do
  end
end
{% endhighlight %}

As you can see, the module contains the `main/1` function wich will take the 
command line arguments map as an argument. Now, to make any sense of the 
arguments, we need to use the `OptionParser` module which comes with Elixir. 
The `OptionParser` 
([docs](http://elixir-lang.org/docs/v1.0/elixir/OptionParser.html)) module 
contains functions which can parse the command line arguments.

If I were to wish the syntax of the arguments of the command line application, 
I'd like to see something like:

{% highlight bash %}
eight_ball --question "Is Elixir great?"
{% endhighlight %}

or 

{% highlight bash %}
eight_ball -q "Is Elixir great?"
{% endhighlight %}

So, let's use `OptionParser` and make `-q/--question` an argument:

{% highlight elixir %}
defmodule EightBall.CLI do
  def main(argv) do
    {options, _, _} = OptionParser.parse(argv, 
      switches: [question: :string],
    )

    IO.inspect options
  end
end
{% endhighlight %}

The `main/1` function will parse arguments and build meaningful tagged lists 
that we can use. At this time, the `main/1` function will only show the options 
that have been parsed. We will add some meaningful logic later. 

First, let's build the executable with `escript`. To build the executable, in the
project root we need to run:

{% highlight bash %}
mix escript.build
{% endhighlight %}

This will compile the Elixir application and build an `escript` executable.

{% highlight bash %}
➜  eight_ball git:(master) ✗ mix escript.build
Compiled lib/eight_ball/cli.ex
Generated eight_ball app
Generated escript eight_ball with MIX_ENV=dev
{% endhighlight %}

If you check the contents of the root path of the `eight_ball` project, you will
see an `eight_ball` executable. To use it, type:

{% highlight bash %}
./eight_ball --question "Is Elixir great?"
{% endhighlight %}

And you will get the following output:

{% highlight bash %}
➜  eight_ball git:(master) ✗ ./eight_ball -q "Is Elixir great?"
[question: "Is Elixir great?"]
{% endhighlight %}

Voila! We can see the parsed command line arguments.

## Wiring it up

Now, let's use the `EightBall.ask/1` function in the CLI app. In the 
`EightBall::CLI.main/1` function, add the following code:

{% highlight elixir %}
defmodule EightBall.CLI do
  def main(opts) do
    {options, _, _} = OptionParser.parse(opts, 
      switches: [question: :string],
      aliases: [q: :question] # makes '-q' an alias of '--question'
    )

    try do
      IO.puts EightBall.ask(options[:question]) 
    rescue 
      e in RuntimeError -> e
        IO.puts e.message
    end
  end
end
{% endhighlight %}

In the try/rescue block we send the question string to the `ask/1` function and
rescue from a `RuntimeError`. This error can be triggered by 
`EightBall::QuestionValidator` which validates the input string. If the input is
not a question, it will throw an error:

> Question must be a string, ending with a question mark.

If our command line application rescues this error, it will output the error 
message.

## Building the CLI application

Now, the last step of building the command line application is invoking escript.
With Elixir, as we saw earlier, it's super easy to invoke `esciprt` via `mix`:

{% highlight bash %}
mix escript.build
{% endhighlight %}

If you are following along, the output of the command should be similar to this:

{% highlight bash %}
➜  eight_ball git:(master) ✗ mix escript.build
Compiled lib/eight_ball.ex
Generated eight_ball app
Generated escript eight_ball with MIX_ENV=dev
{% endhighlight %}

This will create a command line application, with the same filename as the main
module. In our case, this will be just `eight_ball`. Now, if one would open the
executable, there will be a ton of non-readable code. This is due to the fact
that the code you will see is Erlang VM bytecode.

While the code is unreadable, the awesome thing is that you can send this command
line application to anyone that has just Erlang installed on her/his machine. 
Elixir itself is embedded in the file, so the only dependency is Erlang. 
Isn't that cool?

Now, if we run the app:

{% highlight bash %}
➜  eight_ball git:(master) ✗ ./eight_ball --question "Is Elixir awesome?"
Outlook good
{% endhighlight %}

Or

{% highlight bash %}
➜  eight_ball git:(master) ✗ ./eight_ball --question "Is Elixir awesome"
Question must be a string, ending with a question mark.
{% endhighlight %}

### Outro

Thanks for following along this post. I hope you learn something and you consider
it a time well spent. Have you built anything interesting with Elixir? Share it
with me in the comments, I would love to check it out!
