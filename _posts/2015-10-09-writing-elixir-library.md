---
layout: post
title: "Write and publish your first Elixir library"
tags: [elixir, package, mix]
---

As some of you have heard lately, [Elixir](http://elixir-lang.org) is the new hotness. 
Is it just hype? Well, I thought so at first, but I told myself "heck, even if it's 
a waste of time, at least I'll broaden my horizons". Which, if you think about it, it not really is a waste of time. 

Long story short, after couple of weeks of fiddling with the language, mostly 
by playing with it's [web framework](http://phoenixframework.org) I am delighted 
to say that it's a really cool language that you should try out and also, I published 
a really small [API wrapper](https://github.com/fteem/excountries) for Elixir. 

So today, I am going to walk you through writing a simple Elixir library and 
publishing it to Hex.pm, so it will be available for the whole world to use. 

## Got a yes/no question?

I am very sure all of you have heard about the Magic 8 Ball. It's this toy that 
looks like a black and white 8 ball (hence the name). When you ask it a yes/no question
it will (magically) answer the question.

![](http://meme.zenfs.com/u/dcb642467440d773e9fa565a79fe66b42c9f247c.jpeg)

Why don't we create a small Elixir library that one can ask questions to and get 
the question answered? We'll call it `eight_ball`, paying our respect to the 
Magic 8 Ball.

## Let's mix it up!

`Mix` is Elixir's build tool, that allows you to easily create projects, 
manage tasks, run tests and much more. `Mix` also can manage dependencies and 
integrates very nicely with [Hex](http://hex.pm). Hex is a package manager that 
provides dependency resolution and the ability to remotely fetch packages. We will 
publish our project to Hex.pm at the end of this tutorial.

But first, let's create a new Elixir project. Project skeletons are created with 
the command:

{% highlight elixir %}
  mix new <project-name-here>
{% endhighlight %}
Or, in our case:

{% highlight elixir %}
  mix new eight_ball
{% endhighlight %}

When you run the command, you will get an output like this:

{% highlight bash %}
➜  mix new eight_ball
* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/eight_ball.ex
* creating test
* creating test/test_helper.exs
* creating test/eight_ball_test.exs

Your mix project was created successfully.
You can use mix to compile it, test it, and more:

    cd eight_ball
    mix test

Run `mix help` for more commands.
{% endhighlight %}

If you open the project directory with your favourite text editor, you can see 
all of the files that were generated. If you have seen any other Elixir projects, 
the file structure is pretty much the same. 

## What is what?

Just in case anyone gets confused by all of the generated files, let's see what 
is what here:

- README.md -> This is just the README of the project
- .gitignore -> .gitignore file
- mix.exs -> the Mixfile. It's basically the project definition. If you have any experience with Ruby, this is simillar to the .gemspec file
- config -> configuration files directory
- config/config.exs -> Application and it's dependencies configuration 
- lib -> Where the code of our library will live
- lib/eight_ball.ex -> The EightBall module
- test -> Tests directory
- test/test_helper.exs -> Test helper file
- test/eight_ball_test.exs -> The test file for the EightBall module

## The answers

According to [the Wikipedia page](https://en.wikipedia.org/wiki/Magic_8-Ball#Possible_answers) 
for the Magic 8 Ball, the 20 answers inside a standard Magic 8 Ball are:

<small>
1. It is certain<br/>
2. It is decidedly so<br/>
3. Without a doubt<br/>
4. Yes, definitely<br/>
5. You may rely on it<br/>
6. As I see it, yes<br/>
7. Most likely<br/>
8. Outlook good<br/>
9. Yes<br/>
10. Signs point to yes<br/>
11. Reply hazy try again<br/>
12. Ask again later<br/>
13. Better not tell you now<br/>
14. Cannot predict now<br/>
15. Concentrate and ask again<br/>
16. Don't count on it<br/>
17. My reply is no<br/>
18. My sources say no<br/>
19. Outlook not so good<br/>
20. Very doubtful<br/>
</small>

Let's make a list from these answers and add them as a module variable in the 
`EightBall` module.

{% highlight elixir %}
defmodule EightBall do
  # Found at https://en.wikipedia.org/wiki/Magic_8-Ball#Possible_answers
  @answers [
    "It is certain",
    "It is decidedly so",
    "Without a doubt",
    "Yes, definitely",
    "You may rely on it",
    "As I see it, yes",
    "Most likely",
    "Outlook good",
    "Yes",
    "Signs point to yes",
    "Reply hazy try again",
    "Ask again later",
    "Better not tell you now",
    "Cannot predict now",
    "Concentrate and ask again",
    "Don't count on it",
    "My reply is no",
    "My sources say no",
    "Outlook not so good",
    "Very doubtful"
  ]
end
{% endhighlight %}

Our next step will be to introduce a `ask/1` function to our module. It will 
receive the question as a argument and return an answer. Now, we are
faced with a problem. The Magic 8 Ball magically answers questions, because, it's a magic ball.
But, although programming often looks like magic to other people, we know that you can't
program "magic". So, we'll go with a random answer to a question. 

We will know it's not magic, but atleast, it will look like it is!

{% highlight elixir %}
defmodule EightBall do
  def ask(_question) do
    @answers |> Enum.shuffle |> List.first
  end
end
{% endhighlight %}
<small>Note: The `@answers` list is intentionally omitted.</small>

The function takes the `@answers` list, shuffles it using `Enum.shuffle/1` 
([docs](http://elixir-lang.org/docs/stable/elixir/Enum.html#shuffle/1)) and pipes it
into `List.first/1` ([docs](http://elixir-lang.org/docs/stable/elixir/List.html#first/1))
which will take the first item from the shuffled list and return it.

Starting Elixir v1.1, the core team added the `Enum.take_random/2` 
([docs](http://elixir-lang.org/docs/stable/elixir/Enum.html#take_random/2)) function, 
that takes `n` random items from a list.

If you prefer to use v1.1 with `Enum.take_random/2`:

{% highlight elixir %}
defmodule EightBall do
   def ask(_question) do
     @answers |> Enum.take_random(1) |> List.first
   end
end
{% endhighlight %}

## What about the question?

In the `EightBall.ask/1` function, we don't use the actual question. As you 
can see, we ignore the question argument by prepending the argument name with an 
underscore, or `_question`.

Why don't we do something with the question? For example, let's validate it? 
Every question is compiled of words and every question ends with e a question mark. 
So, we want a string as an argument, which ends with a question mark. 
Any other type of argument should be ignored. Or, rather, inform the user of our 
library that it expects a question.

Let's add some validation to our little `ask/1` function. I will wish the
interface of the validator first and we'll write the implementation after that.

{% highlight elixir %}
defmodule EightBall do
  def ask(question) do
    EightBall.QuestionValidator.validate!(question)
    @answers |> Enum.shuffle |> List.first
  end
end
{% endhighlight %}

Now, the `EightBall.QuestionValidator.validate!/1` function will take the
question as an argument and throw an error if the question does not have the format
we expect.

## QuestionValidator

Let's write our simple validator. 

{% highlight elixir %}
defmodule EightBall.QuestionValidator do
  # Question must end with a '?'
  @validation_regex ~r/\?$/

  def validate!(question) when is_binary(question) do
    unless String.strip(question) =~ @validation_regex, do: throw_validation_error
  end

  def validate!(question) do
    throw_validation_error 
  end

  defp throw_validation_error do
    throw "Question must be a string, ending with a question mark."
  end
end
{% endhighlight %}

Looking at the code, top to bottom, there are couple of key points. First, the
`@validation_regex` variable, is a regular expression. It will match any 
strings that end with a question mark. 

Second, the `validate!/1` function. The first function clause will match 
when the question is a binary. Why binary? Well, Elixir uses UTF8 encoding for
string, which basically makes [strings UTF8 encoded binaries](http://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html#utf-8-and-unicode). If the first function clause matches, 
it will strip the unnecessary whitespace using `String.strip/1` and match the
`@validation_regex`. If it does not match, it will throw an error.

Also, if the first function clause does not match, it will throw a validation error.

## Testing with IEx

Let's see how our library works in IEx (Interactive EliXir). In the project directory,
run:

{% highlight bash %}
iex -S mix
{% endhighlight %}

IEx will compile and load our library in the IEx session, so we can start using it right away.
Try running:

{% highlight bash %}
iex(1)> EightBall.ask("Can I fly?")
"Without a doubt"
{% endhighlight %}

When I asked a question, the library returned "Without a doubt". Nice! Let's 
check our validation:

{% highlight bash %}
iex(2)> EightBall.ask("This should fail.")
** (throw) "Question must be a string, ending with a question mark."
    (eight_ball) lib/eight_ball/question_validator.ex:14: EightBall.QuestionValidator.throw_validation_error/0
    (eight_ball) lib/eight_ball.ex:27: EightBall.ask/1
{% endhighlight %}

When we sent a statement instead of a question, the library threw an error. Good.
Another (and better) way to test this is by writing actual tests, but, we'll leave 
that for another day.

## Publishing `EightBall` to Hex

Hex has a very good [documentation](https://hex.pm/docs/publish) on publishing packages.
If you want to read and understand the details, head over to the documentation. Here,
we'll just cover the basic steps needed to publish this libraty to Hex.pm.

### Registration

If you do not have a user registered, you can do it via the command line:

{% highlight bash %}
mix hex.user register
{% endhighlight %}

Hex will prompt for your username, email and password and then it will create an 
API key that will be stored in the `~/.hex` directory.

### Defining the package

The package is configured in the `project` function in the project's `mix.exs` file.
Let's create a private function in our `EightBall.Mixfile` module and call it 
`package`:

{% highlight elixir %}
defp package do
  [
    files: ["lib", "mix.exs", "README", "LICENSE*"],
    maintainers: ["Ilija Eftimov"],
    licenses: ["Apache 2.0"],
    links: %{"GitHub" => "https://github.com/fteem/eight_ball"}
  ]
end
{% endhighlight %}

These properties will define the files of the package and some metadata like
maintainers' name and licences. Then, in the `EightBall.Mixfile.project/0` 
function, we need to include the package definiton:

{% highlight elixir %}
defmodule EightBall.Mixfile do
  use Mix.Project

  def project do
    [app: :eight_ball,
     version: "0.0.1",
     elixir: "~> 1.0",
     build_embedded: Mix.env == :prod,
     start_permanent: Mix.env == :prod,
     deps: deps,
     package: package ]
  end

  *snip*
end
{% endhighlight %}

Also, make sure the version is set properly. In our case, we can leave it as
`0.0.1`. The last step after you are happy with the package definition is the 
publish step itself. Publishing a package is done by running:

{% highlight bash %}
➜  eight_ball git:(master) mix hex.publish
Publishing eight_ball v0.0.1
  Dependencies:
  Files:
    lib/eight_ball.ex
    lib/eight_ball/question_validator.ex
    mix.exs
    README.md
    LICENSE
  App: eight_ball
  Name: eight_ball
  Version: 0.0.1
  Build tools: mix
  Description: Library that acts like a real life Magic 8 Ball.
  Licenses: Apache 2.0
  Maintainers: Ilija Eftimov
  Links:
    GitHub: https://github.com/fteem/eight_ball
  Elixir: ~> 1.0
Before publishing, please read Hex Code of Conduct: https://hex.pm/docs/codeofconduct
Proceed? [Yn] y
[#########################] 100%
Published at https://hex.pm/packages/eight_ball/0.0.1
Don't forget to upload your documentation with `mix hex.docs`
{% endhighlight %}

Easy as that. After publishing your package, it's available at it's 
[Hex.pm page](https://hex.pm/packages/eight_ball).

## Outro

That's it. As you can see, publishing the library to Hex.pm is quite straight forward. 
Also, generating the project skeleton is really easy with Mix. All it takes is one
command and the project is set up.

If you are looking for more challenge, you can add some tests for the validator 
module. Also, adding documentation is quite important, so if you feel like writing 
some documentation - do it! But, whatever you decide to do, go and share your library with the world. 
Push it to Hex.pm and make it available for everyone to download and use.

Did you follow along this tutorial? Did you have any hiccups or did it all go smooth? 
Let me know in the comments below!

<small>P.S. You can see the code for this post [here](https://github.com/fteem/eight_ball). 
The Hex package page is [here](https://hex.pm/packages/eight_ball).</small>

