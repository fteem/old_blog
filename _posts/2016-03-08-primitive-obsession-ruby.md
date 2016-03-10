---
layout: post
title: "Refactoring in Ruby: Primitive Obsession"
tags: [refactoring, ruby, primitive, obsession]
image: primitive-obsession.png
---

We've all been at this point where we have bloated our classes with primitive
values all over the place. Usually, we drop in primitive constants that, for 
whatever reason, we think that are a good fit to the class. Or sometimes, we just
dump primitive values instead of small objects, thinking "it's okay, it's just
an attribute in the class". But, does it always make sense?

## The problem

Say we have a project for a finance journalist who wants us to automate his text 
editor to do some random fixes to his texts, on the fly. The automation is 
basically adding his name and date to the footer of the text that he's writing, 
and to change currencies abbreviations to symbols. The good thing for us is that
his editor is written in Ruby, and has the ability to add custom I/O streams to
it's I/O stack.

So, our specification for this I/O stream, is:

- limit text to no more than 10000 words
- append journalist name to end of article
- swap popular currency names abbreviations with symbols

After a bit of work, we come up with this quick hack for a I/O stream:

{% highlight ruby %}
require 'stringio'

class FinanceIO
  class MaximumLengthExceeded < StandardError; end

  MAX_WORDS = 10000
  MIN_WORDS = 2000
  AUTHOR_NAME  = "John Smith"
  AUTHOR_EMAIL = "j.smith@nytimes.com"
  CURRENCIES = [
    ["USD", "$"],
    ["GBP", "£"],
    ["JPY", "¥"],
    ["CHF", "₣"],
    ["EUR", "€"],
    ["INR", "₹"],
    ["GEN", "¤"]  # General currency sign
  ]
  
  def initialize
    @io = StringIO.new
  end

  def puts(arg)
    if @io.string.split("").length >= MIN_WORDS && @io.string.split("").length =< MAX_WORDS
      raise MaximumLengthExceeded.new("You've reached the maximum length.")
    else
      CURRENCIES.each do |abbrv, symbol|
        arg.gsub!(abbrv,symbol)
      end
      @io.puts(arg)
    end
  end

  def on_exit
    append_author
  end

  def method_missing(method_name, *args, &block)
    if @io.respond_to?(method_name)
      @io.send(method_name, *args)
    else
      super(method_name, *args, &block)
    end
  end

  private

  def append_author
    @io.puts("\n")
    @io.puts(AUTHOR_NAME)
    @io.puts("\n")
    @io.puts(AUTHOR_EMAIL)
    @io.puts("\n")
  end

end
{% endhighlight %}

The `FinanceIO` class is a wrapper around a `StringIO`, which overrides a couple
of methods, but relays any other method calls to the wrapped `StringIO` object.
This allows us to do our work easily, while not breaking when any object of the
I/O stack tries to call a built-in method on our object.

Let's break down the example. The `MAX_WORDS` and `MIN_WORDS` is a limitation 
that our client (the journalist) has been given by the newspaper he works for, 
or in other words, his articles cannot be loger than 10000 words. The
`AUTHOR_NAME` constant is the name of our author. We don't need to enable 
customization for this piece of data, because our client wanted to make our life 
easier by agreeing that he will not change his name any time soon. The 
`AUTHOR_EMAIL` constant is the author public email address. Next, the 
`CURRENCIES` constant is a two-dimensional array which contains the abbreviations 
and the symbols of the most popular currencies that the journalist uses while 
writing his finance articles. 

Most of the functionality described in the spec is done within the 
`FinanceIO#puts` method:

{% highlight ruby %}
def puts(arg)
  if @io.string.split("").length >= MIN_WORDS && @io.string.split("").length =< MAX_WORDS
    raise MaximumLengthExceeded.new("You've reached the maximum length.")
  else
    CURRENCIES.each do |abbrv, symbol|
      arg.gsub!(abbrv,symbol)
    end
    @io.puts(arg)
  end
end
{% endhighlight %}

The `puts` method takes a string as an argument, and saves it to the stream. But,
before it does that, it checks if the text is longer than 10000 words. If it is,
it will raise an exception that the text editor will handle gracefully. If not,
it will carry on to substitute any of the currency abbreviations in the text with
the corresponding symbol.

Also, the `FinanceIO#on_exit` is a method which will append the author name 
in the footer of the text. 

{% highlight ruby %}
def on_exit
  append_author
end
{% endhighlight %}

This will get automatically invoked by the editor, as an callback before the 
journalist closes his editor.

This will surely work, and it complies with our imaginary text editor. Sure,
it might be a bit dirty, but it does the trick. Now, a question to ourselves -
what should we do about the data in the constants? There seems to be much 
behaviour in connection with these constants. Aside of that, it's just plain data
and it doesn't really add too much noise to our class and the functionality is
easy to follow... right?

## The Code Smell

Well, sure, the fields don't do much noise, but this is exactly the way of 
thinking that creates these code smells. You could argue that the `FinanceIO` 
class is pretty simple to follow, and you would be right. The problem lies in the
practice, which will eventually bloat the class with so much "static" data,
to a point of total confusion. Sure, our journalist could be happy with our hack,
which is great, but he will surely come back one day with more specs for 
customizations. That's what usually happy customers do - they return for more
stuff.

Although this code smell is not that dangerous at first, it **will** cause havoc 
in the near future. The **Primitive Obsession** code smell is one of the simplest 
ones. It can be spotted in classes where primitives are used to "simulate" types. 
This means that instead of separating a data type for the primitives, we keep 
some sort of a primitive value, whether it's a string, or a number or an array of 
any values. As a next step, we usually add simple names to these primitive values 
(see our class above), and we keep on thinking that it is very clear what these 
values do. 

Another occurance of this code smell is creating a "field simulation". This 
happens when we add an array of primitive values to the class, which contains
some data that is used in the class logic, somewhere.

Sure, three to five values are easy to track. What happens when you get to 
twenty? Thirty maybe? Yeah, I have seen some classes where these primitive values
are all over the place, and let me tell you, it's no fun!

So, how can we avoid and/or detect this code smell? Let's see!

## Refactoring

When it comes to refactoring this code smell, you can take multiple paths. For
example, if you have a large variety of these primitives, you can always look for 
logical links between them. Another way is, if these primitive values are used as 
method parameters, we could introduce a parameter objects, or we could change 
arrays into objects as well. There are more forms of this code smell, but let's 
tackle them one by one and see the approaches to refactoring.

### Logical linking

Like we saw earlier, often these primitive values have some logial link between
them. Although our list of primitives is not that long, you can see the logical 
link between `AUTHOR_NAME` and `AUTHOR_EMAIL`, right? 

Since the logical link between these two primitives and the 
`FinanceIO#append_author` method are easily noticable, we know that these two 
constants and the method should be grouped in some way. Here, we can refactor the 
values with an object, which will be called `Author`:

{% highlight ruby %}
class Author
  attr_reader :first_name, :last_name, :email
  def initialize(first_name = "John", last_name = "Smith", email = "j.smith@nytimes.com")
    @first_name = first_name
    @last_name = last_name
    @email   = email
  end

  def full_name
    last_name + " " + first_name
  end
end
{% endhighlight %}

Pretty straight forward. Now, if you think about it, you can notice that the
`FinanceIO#append_author` is basically the signature of the journalist. So, we
can extract that method into our new `Author` class:

{% highlight ruby %}
def signature
%Q{
  #{full_name}
 
  #{email}
}
end
{% endhighlight %}

Now, having the behaviour and the values extracted to a separate class, we can
refactor `FinanceIO` a bit:

{% highlight ruby %}
require 'stringio'

class FinanceIO
  class MaximumLengthExceeded < StandardError; end

  MAX_WORDS = 10000
  MIN_WORDS = 2000
  CURRENCIES = [
    ["USD", "$"],
    ["GBP", "£"],
    ["JPY", "¥"],
    ["CHF", "₣"],
    ["EUR", "€"],
    ["INR", "₹"],
    ["GEN", "¤"]  # General currency sign
  ]
  
  def initialize
    @io = StringIO.new
    @author = Author.new
  end

  def puts(arg)
    if @io.string.split("").length >= MIN_WORDS && @io.string.split("").length =< MAX_WORDS
      raise MaximumLengthExceeded.new("You've reached the maximum length.")
    else
      CURRENCIES.each do |abbrv, symbol|
        arg.gsub!(abbrv,symbol)
      end
      @io.puts(arg)
    end
  end

  def on_exit
    @io.puts(@author.signature)
  end

  def method_missing(method_name, *args, &block)
    if @io.respond_to?(method_name)
      @io.send(method_name, *args)
    else
      super(method_name, *args, &block)
    end
  end

end
{% endhighlight %}

Much cleaner, right? We have the values and the behaviour extracted to a separate 
class, and the `Author#signature` method clearly shows that the behaviour
revolving the author signature should be a part of the `Author` class.

Now, since the first name, last name and email are set to default parameters on
the `Author#initialize` method, we could go a step further and extract them to
a `DefaultAuthor` subclass:

{% highlight ruby %}
class DefaultAuthor < Author
  def initialize
    super("John", "Smith", "j.smith@nytimes.com")
  end
end
{% endhighlight %}

Having this class, we can easily customize our `FinanceIO` stream and use a
non-default author just by tweaking the constructor by using dependency 
injection:

{% highlight ruby %}
class FinanceIO
  # snipped..

  def initialize(author = DefaultAuthor.new)
    @io     = StringIO.new
    @author = author
  end

  # snipped..
end
{% endhighlight %}

### Replacing the array with... something meaningful

When we got to this step I assume you were wondering what we will do with the
`CURRENCIES` two-dimensional array. Well, let's disect the array a bit and see
what we are actually dealing with.

Well, just like the array name states, we are dealing with a bunch of currencies
here, ergo, we need a `Currency` class to encapsulate the abbreviation and the
symbol:

{% highlight ruby %}
class Currency
  attr_reader :abbreviation, :symbol
  def initialize(abbreviation, symbol)
    @abbreviation = abbreviation
    @symbol       = symbol
  end
end
{% endhighlight %}

So, how can we eliminate the array from the `FinanceIO` class and move it 
somewhere else? Let's start moving the array step by step - first, by extracting
it into the `Currency` class. We will move the collection and we will also add a
utility method `Currency.all` which will return a collection of all the 
currencies, as `Currency` objects:

{% highlight ruby %}
class Currency
  DEFAULT_CURRENCIES = [
    ["USD", "$"],
    ["GBP", "£"],
    ["JPY", "¥"],
    ["CHF", "₣"],
    ["EUR", "€"],
    ["INR", "₹"],
    ["GEN", "¤"] 
  ]

  def self.all(currencies_list = DEFAULT_CURRENCIES)
    currencies_list.collect do |abbr, sym|
      new(abbr, sym)
    end
  end

  attr_reader :abbreviation, :symbol
  def initialize(abbreviation, symbol)
    @abbreviation = abbreviation
    @symbol       = symbol
  end
end
{% endhighlight %}

Now, refactoring the `FinanceIO#puts` method should be straight forward:

{% highlight ruby %}
def puts(arg)
  if @io.string.split("").length >= MIN_WORDS && @io.string.split("").length =< MAX_WORDS
    return MaximumLengthExceeded.new("You've reached the maximum length.")
  else
    Currency.all.each do |currency|
      arg.gsub!(currency.abbreviation, currency.symbol)
    end
    @io.puts(arg)
  end
end
{% endhighlight %}

Sure, this could work, but we still have a bit of polution in the `Currency`
class. What we could do, instead of keeping the currencies as a constant, we
could move them to a method, or better yet, move them to a new class, with a
method that would return all of them:

{% highlight ruby %}
class DefaultCurrencies
  def self.all
    [
      ["USD", "$"],
      ["GBP", "£"],
      ["JPY", "¥"],
      ["CHF", "₣"],
      ["EUR", "€"],
      ["INR", "₹"],
      ["GEN", "¤"] 
    ]
  end
end
{% endhighlight %}

Then, adding this class as a depenency to the `Currency` class will be pretty
easy. But also, **uncoupling** it will be also easy, if we decide to move these
hardcoded primitive values to a database:

{% highlight ruby %}
class Currency
  def self.all
    DefaultCurrencies.all.collect do |abbr, sym|
      new(abbr, sym)
    end
  end

  attr_reader :abbreviation, :symbol
  def initialize(abbreviation, symbol)
    @abbreviation = abbreviation
    @symbol       = symbol
  end
end
{% endhighlight %}

With this step we have encapsulated our collection of data, and made it easy to
replace in next iteration of our program. Our `Currency` class only depends
on a class method, which would return a two-dimensional array with the currencies
data. With this in place, we can come back to the `FinanceIO` class and polish it 
up.

### Introducing a parameter object

Let's look at the state of our `FinanceIO` class now. The only leftovers from our
refactoring are the `MAX_WORDS` and `MIN_WORDS` primitives. Their purpose is to 
limit the length of the text that the author can write. Let's see how we can
refactor it out of the class.

If you look at the `FinanceIO#puts` method, you can notice that these constants 
are used only as a parameter, to check if the length words are within the limit. 

Having this in mind, we can extract a **Parameter Object**. Parameter objects
are usually like containers for data that has same or similar role. Our 
limitation primitives are a nice example - they do not belong to the `FinanceIO`
class, but their only purpose is to validate the input of the user. Let's
extract them out to a separate class:

{% highlight ruby %}
class EditorialLimitations
  def initialize
    @max_words = 10000
    @min_words = 2000
  end

  def within?(text)
    text_length = words(text)
    text_length >= @min_words && text_length =< @max_words
  end

  private

  def words(txt)
    txt.split("").length
  end
end
{% endhighlight %}

As you can see, by extracting this parameter object, we are even able to extract
the logic which will validate the text of the journalist. Now, let's use our
newly created class in the program:

{% highlight ruby %}
require 'stringio'

class FinanceIO
  class MaximumLengthExceeded < StandardError; end

  def initialize
    @io          = StringIO.new
    @author      = Author.new
    @limitations = EditorialLimitations.new
  end

  def puts(arg)
    if @limitations.within?(io.string)
      Currency.all.each do |currency|
        arg.gsub!(currency.abbreviation, currency.symbol)
      end
    else
      raise MaximumLengthExceeded.new("You've reached the maximum length.")
    end
    @io.puts(arg)
  end

  def on_exit
    @io.puts(@author.signature)
  end

  def method_missing(method_name, *args, &block)
    if @io.respond_to?(method_name)
      @io.send(method_name, *args)
    else
      super(method_name, *args, &block)
    end
  end

end
{% endhighlight %}

As you can see, although there's still a lot going on in the `FinanceIO#puts`
method, it's much easier to understand what's going on. Also, if we would write
tests around this code, testing would be much easier. 

## Outro

We could do more refactoring here, especially in the `FinanceIO#puts` method, but
for the scope of this article we will stop there. I encourage you to carry on 
with refactoring the code, and writing some tests - practice makes it perfect,
doesn't it?

Until next time!
