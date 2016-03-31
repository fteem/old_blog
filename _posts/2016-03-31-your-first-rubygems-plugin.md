---
layout: post
title: "Write your first Rubygems plugin"
tags: [ruby, gems, rubygems, plugin]
image: write-rubygems-plugin.png
---

I don't think that at this point Rubygems needs any introduction. Except if 
you have been living under a rock for the last 10 years or so. In that case,
I think that you wouldn't be here reading this blog. You would be having a 
problem understanding why someone would like to share what they are eating, or
what they are doing at the moment. For the rest of you, have you heard that 
Rubygems is extensible? Let's see how Rubygems does this, and how we can make
our own Rubygem plugin.

## Extensibility

Rubygems, since version 1.3.2 has the ability to load plugins installed in the
gems collection or the `$LOAD_PATH`. This means that you can create a Ruby program
that will extend Rubygems with your own logic. The best way to go about this is
by packing it into a gem. It gets a bit of *meta*, I know. Basically, what I am
saying is that you can have a gem which will extend Rubygems, when installed.

Now, there are is a naming convention when it comes to plugins. They must be 
named `rubygems_plugin` and placed at the root of your gem’s `#require_path`. 
Plugins are discovered via `Gem::find_files` and then loaded. But, instead of
dwelling into the theory, why don't we go ahead and build one ourselves?

## Taken?

No, not this guy!

![](http://e.lvme.me/o3rw8jl.jpg)

We are going to keep it peaceful here, no guns, kidnappers and violence. We will
continue talking about gems. Any time we want to create a new gem, we are in a
need of a name. Let's make a Rubygems plugin that will add a command to check 
if a gem name is available. The desired command will be:

```bash
gem taken liam
```

And the output can vary between

```txt
The gem name 'liam' is taken.
```

and

```txt
It will be yours. Oh yes. It will be yours. The gem name 'liam' is available.
```

And you guess it - the plugin's name will be `taken`.

## Setting the foundations

Let's create a new gem, using the command:

```bash
bundle new taken
```

This will create a new barebones gem, called `Taken`:

{% highlight bash %}
create  taken/Gemfile
create  taken/.gitignore
create  taken/lib/taken.rb
create  taken/lib/taken/version.rb
create  taken/taken.gemspec
create  taken/Rakefile
create  taken/README.md
create  taken/bin/console
create  taken/bin/setup
create  taken/.travis.yml
create  taken/test/test_helper.rb
create  taken/test/taken_test.rb
create  taken/LICENSE.txt
create  taken/CODE_OF_CONDUCT.md

Initializing git repo in /some/path/here/taken
{% endhighlight %}

First, let's do a bit of restructuring of our gem files and directories, before
we start with writing some code. In the `lib` directory, we will need to add
the `rubygems_plugin.rb` file, which will be picked up by Rubygems and will load
our new command. Next, in the same directory, we will need to create a new
directory with the name `rubygems`. Here, we will store all of the needed files
for this particular Rubygems plugin. 

This is what our lib directory should look like:

```bash
➜  ls lib
rubygems/  rubygems_plugin.rb
```

That's all. Let's open the `rubygems_plugin.rb` and start writing our new 
command.

## Defining the new command

First, in the plugin file we will need to register our new command. Rubygems
has this utility class, called `CommandManager`. It registers and installs all 
the individual sub-commands supported by the gem command. If you would like to
register a new command against the `Gem::CommandManager` instance, you need to
do the following:

```ruby
require "rubygems/command_manager"

Gem::CommandManager.instance.register_command(:taken)
```

This will tell Rubygems that there's a new command in town, called `taken`. This
also expects that the command program will be stored in the
`lib/rubygems/commands/taken_command.rb` file. Let's create it!

{% highlight ruby %}
class Gem::Commands::TakenCommand < Gem::Command
end
{% endhighlight %}

There are couple of key things when it comes to Rubygems plugins. The first
would be that each command class **should** inherit from the `Gem::Command` 
class. If you look inside the actual source code of the `Gem::Command` class,
you will notice the following comment:

```ruby
# Base class for all Gem commands.  When creating a new gem command, 
# define #initialize, #execute, #arguments, #defaults_str, 
# #description and #usage (as appropriate)...
```

This means that, in our new command we will need to implement the required
methods. Let's begin by adding the initialize method:

{% highlight ruby %}
def initialize
  super("taken", "Checks if the gem name is taken.")
end
{% endhighlight %}

Since our `taken` command will not need any special configurations, we can
keep the initialize method short and sweet. As the first argument to the
initializer, we pass the command name - in our case `taken`. The second argument
is the short summary of the command. 

The next method that we will need to override is the `arguments` methods. This
method is pretty simple - it's just used to provide details of the arguments that
a command will take. In our case, that's the gem name, whose name we would
like to check. Our implementation of the method would look like:

{% highlight ruby %}
def arguments # :nodoc:
  "GEMNAME       gem name to check"
end
{% endhighlight %}

Next is the `description` method. All it does is return a string containing a
longer description of what the command will do.

{% highlight ruby %}
def description
  <<-EOF
    The `taken` command gets all of the available gems names and check if a gem name is free or taken.
  EOF
end
{% endhighlight %}

These are the only methods that we need at the moment. The last one is the
`execute` method, which will do all of the work in our new Rubygems command.

## Implementing `execute`

This method will do most of the heavy lifting in our plugin. We can separate the
flow of the command in couple of steps:

1. Get the gem name from the command arguments,
2. Throw some kind of an error if it's blank or has multiple gem names,
3. Fetch the latest list of gems,
4. Find out if a gem with the name already exists, and
5. Depending on the former step, inform the user for the result

First, let's take the gem name from the command arguments. If you took a bit of
a deeper look in the `Gem::Command` class, I am sure you've noticed the
`Gem::Command#get_one_gem_name` method. This method will take care of the first
two steps of the flow. The actual source code is the following:

{% highlight ruby %}
##
# Get a single gem name from the command line.  Fail if there is no gem name
# or if there is more than one gem name given.

def get_one_gem_name
  args = options[:args]

  if args.nil? or args.empty? then
    raise Gem::CommandLineError,
          "Please specify a gem name on the command line (e.g. gem build GEMNAME)"
  end

  if args.size > 1 then
    raise Gem::CommandLineError,
          "Too many gem names (#{args.join(', ')}); please specify only one"
  end

  args.first
end
{% endhighlight %}

Nice and easy. Let's add invoke this method in our `execute` method:

{% highlight ruby %}
def execute
  gem_name = get_one_gem_name
end
{% endhighlight %}

The next step is to get the latest list of gems. Rubygems has a class called
`Gem::SpecFetcher`, that handles metadata updates from remote gem repositories.
Simply said, this class with retrieve gem specifications and allow you to work
with them. Now, under the hood, the `Gem::SpecFetcher` uses a class called
`Gem::RemoteFetcher`. This class handles communication with remote sources, like
Rubygems.org. It knows how to fetch gem details and info.

Since we would like to have our command to work with the gem specs, we need to
introduce a `fetcher` to our `execute` method and utilize the
`Gem::SpecFetcher#detect` method.

{% highlight ruby %}
def execute
  gem_name = get_one_gem_name

  fetcher = Gem::SpecFetcher.fetcher
  spec_tuples = fetcher.detect do |name_tuple|
    gem_name == name_tuple.name
  end
end
{% endhighlight %}

The `spec_tuples` will be a collection of objects from the `Gem::NameTuple` 
class. If you open it's source code, you can see that it's a utility class that
represents a gem, in a certain format. The explanation within the class is the
following:

{% highlight ruby %}
# Represents a gem of name +name+ at +version+ of +platform+. These
# wrap the data returned from the indexes.
{% endhighlight %}

Since each `Gem::NameTuple` object has a `name` method, we compare the gem name
that was provided by the user with the name from the gem spec. If it matches,
that means that there's already a gem with that name.

{% highlight ruby %}
def execute
  gem_name = get_one_gem_name

  fetcher = Gem::SpecFetcher.fetcher
  spec_tuples = fetcher.detect do |name_tuple|
    gem_name == name_tuple.name
  end

  if name_free?(spec_tuples)
    $stdout.puts "It will be yours. Oh yes. It will be yours. The gem name '#{name}' is available."
  else
    $stdout.puts "The gem name '#{name}' is taken."
  end

  private

  def name_free?(spec)
    !spec.flatten.length.zero?
  end
end
{% endhighlight %}

This is it. We check if the the spec_tuples length is not zero and we show the
user the appropriate message.

## Beyond the command

If you have been following this small tutorial, I am sure you are starting to
get an idea of how you can utilize Rubygems' API to build a plugin. You could
pack all of this logic into a gem, add some tests and publish it. Then people
can install the gem and use the newly created `taken` command.

But, this command is just scratching the surface of the possibilities that 
Rubygems offers. And as a side-win, the Rubygems code is really great. In the
Git logs you can see names that are well respected in the Ruby community. You can
learn how they've modeled their exceptions. Or maybe, how they do the security or
the HTTP layer. 

Have you ever done any contributions to Rubygems? Or maybe you have written a 
Rubygems plugin? Or maybe, you would like to write one but you have run out of
ideas? Hit me up in the comments!

