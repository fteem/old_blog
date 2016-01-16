---
layout: post
title: "Refactoring in Ruby: The right place for a Builder?"
tags: [refactor, ruby, builder]
---

Recently I started tackling refactoring in Ruby with my blog posts. It seems
that it's one of the most popular topics that people would like to read about,
so here I am with another installment of refactoring in Ruby. This time, we will
see if it's the right time and place for a Builder... whatever that means.

## Just a bit of backstory

In my last post on refactoring in Ruby, I covered how to detect and refactor the
[long parameters](/refactoring-smelly-parameters-lists) code smell. It stirred
up quite a discussion in the comments, but also, on my Skype chat. So, a
colleague of mine, who I respect a lot, mentioned that eliminating the long
parameters lists can be done via the builder pattern. I won't go into too much
detail of how the discussion went, but I'll tell you that we had a bit of a
disagreement.

Eventually we shared some thoughts and got on the same page. So, in this blog
post I would like to show you what is the Builder Pattern and what's a good
place to use it.

## The basics

First, the basics - what is the Builder Pattern? The builder pattern is an
object creation design pattern. This means that the code produced when
following this pattern will create objects of certain class(es).

Now, the key thing to the Builder Pattern, which differentiates it from the
Factory pattern, is it's purpose. The Builder pattern is used to apply any sort
of **configuration on object creation**.

This pattern also allows finer control over the object creation. Via the
builder's interface we create an uniformed way of object creation, instead of
dealing with complex flows of creating an object (which often can be a
composite).

## Long parameters lists

Now, back to that topic on the long parameters lists code smell. Does it make
sense to apply the Builder pattern to this type of a code smell? Well, it seems
like it does, but only to a special type of methods - constructors!

If you know just a bit of OOP, you know that constructors are the methods which
are invoked when an object is being created. Now, if a constructor has a long
list of parameters, that usually means that it is doing quite a bit of
configuration of the object itself.

Take a look at this example:

{% highlight ruby %}
class Touchscreen
  # Some nice things a touch screen can do...
end

class CPU
  # Power... unlimited power! #StarWars
end

class RAM
  # Yeah!
end

class Smartphone
  attr_accessor :model, :screen, :cpu, :ram, :memory, :os

  def initialize(model = nil, screen = nil, cpu = nil, ram = nil, memory = nil, os = nil)
    @model  = model
    @screen = screen
    @cpu    = cpu
    @ram    = ram
    @memory = memory
    @os     = os
  end
end

{% endhighlight %}

This is quite simlpe, but it implies that you will need to create objects from
the `CPU` and `Touchscreen` classes, before you create a `Smartphone` object.
It means that you need some configuration done before you can create an actual
`Smartphone`.

As you can see, a big parameters lists can lead to confusion. If in the source
code of the application you will only create `Smartphone` objects couple of
times, then refactoring this is an overkill. But, if the class with the long
parameters list in the constructor is used often, then streamlining the object
creation will be an immediate win.

Before applying the Builder Pattern here, we need to detect some patterns in our
code. By code, I mean production code **and** tests. If you often find yourself
forgetting to create a `CPU` object before assigning it to a `Smartphone`, then
a builder can help with all of that. Basically, a builder should have utility
methods that will make the configuration of these objects easier.

{% highlight ruby %}
class SmartphoneBuilder
  def initialize
    @smartphone = Smartphone.new
  end

  def set_model(model)
    @smartphone.model = model
  end

  def add_processor(speed)
    @smartphone.cpu = CPU.new(speed)
  end

  def add_touchscreen(size)
    @smartphone.screen = Touchscreen.new(size)
  end

  def add_ram(amount)
    @smartphone.ram = RAM.new(amount)
  end

  def add_memory(amount)
    @smartphone.memory = amount
  end

  def set_os(os)
    @smartphone.os = os
  end

  def smartphone
    obj = @smartphone.dup
    @smartphone = Smartphone.new
    return obj
  end
end
{% endhighlight %}

Having these methods in our `SmartphoneBuilder` class will improve the object
creation flow. Take this dummy code for a comparison.

Before:

{% highlight ruby %}
class Order
  attr_accessor :products
  def initialize
    @products = []
  end
end

class OrderTest < Minitest::Test
  def test_contains_multiple_products
    order = Order.new
    5.times do
      order.products << Smartphone.new("Apple iPhone 6S", 4.7, 1.84, 2048, 16384, :ios)
    end
    assert_equal 5, order.products.length
    # Sanity check
    assert_equal "Apple iPhone 6S", order.products.last.model
  end
end
{% endhighlight %}

Just writing the line

{% highlight ruby %}
order.products << Smartphone.new("Apple iPhone 6S", 4.7, 1.84, 2048, 16384, :ios)
{% endhighlight %}

takes effort, because it's nearly impossible to remember the order of the
parameters. Meta-note: When I was writing the example, I had to copy-paste the initializer
arguments and fill them up one-by-one, so I don't mix them up.

Now, writing the same test with the builder:

{% highlight ruby %}
class OrderTest < Minitest::Test
  def test_contains_multiple_products
    order = Order.new
    smartphone_builder = SmartphoneBuilder.new
    5.times do
      smartphone_builder.set_model("Apple iPhone 6S")
      smartphone_builder.add_touchscreen(4.7)
      smartphone_builder.add_processor(1.84)
      smartphone_builder.add_ram(2048)
      smartphone_builder.add_memory(16384)
      smartphone_builder.set_os(:ios)
      order.products << smartphone_builder.smartphone
    end
    assert_equal 5, order.products.length
    # Sanity check
    assert_equal "Apple iPhone 6S", order.products.last.model
  end
end
{% endhighlight %}

As you can see, although the example is quite contrived, introducing the builder
pattern comes in handy. Sure, the second code example might be more verbose, but
on the other hand it's super clear and very hard to get confused by it. There
are a ton of other situations where one could use the builder pattern, but in
this case it plays nicely when eliminating this code smell.

**Edited**, 16th of January, 2016:

Additionally, thanks to couple of readers and colleagues, who I **thank a ton**
for weighing in, a very interesting and approach to builders is using a building
block. The usage would look like:

{% highlight ruby %}
class OrderTest < Minitest::Test
  def test_contains_multiple_products
    order = Order.new
    5.times do
      smartphone = SmartphoneBuilder.build do |builder|
        builder.set_model("Apple iPhone 6S")
        builder.add_touchscreen(4.7)
        builder.add_processor(1.84)
        builder.add_ram(2048)
        builder.add_memory(16384)
        builder.set_os(:ios)
      end
      order.products << smartphone
    end
    assert_equal 5, order.products.length
    # Sanity check
    assert_equal "Apple iPhone 6S", order.products.last.model
  end
end
{% endhighlight %}

As you can see, this looks super nice, which is half the reason I agreed to add
this to the post after it was published. The other half is usefulness, obviously.
The implementation of the block-style builder:

{% highlight ruby %}
class SmartphoneBuilder
  def self.build
    builder = new
    yield(builder)
    builder.smartphone
  end

  def initialize
    @smartphone = Smartphone.new
  end

  def set_model(model)
    @smartphone.model = model
  end

  def add_processor(speed)
    @smartphone.cpu = CPU.new(speed)
  end

  def add_touchscreen(size)
    @smartphone.screen = Touchscreen.new(size)
  end

  def add_ram(amount)
    @smartphone.ram = RAM.new(amount)
  end

  def add_memory(amount)
    @smartphone.memory = amount
  end

  def set_os(os)
    @smartphone.os = os
  end

  def smartphone
    obj = @smartphone.dup
    @smartphone = Smartphone.new
    return obj
  end
end
{% endhighlight %}

By implementing the `SmartphoneBuilder.build` method, we expose the additional
block-style variant of the builder, so the user can choose between the
good-looking block building or the not-so-good-looking non-block-style version.

## (Over) Validation

Earlier we agreed that as an object creational patten, the Builder comes into
play when there's some configuration heavy lifting when creating objects. Well,
another place where a Builder might fit in nicely is validation-on-creation.

There have been plenty of cases where we want to make sure that the data
that is passed in on object creation is valid. Like, a smartphone without a
touchscreen is not really a smartphone, is it? Well, at least for now.

Anyway, the Builder Pattern comes nicely into play when we want to deal with
validation. Since the data is pushed into the object in it's constructor, the
most logical place to do the validation is there, in the constructor:

{% highlight ruby %}
class Smartphone
  attr_accessor :model, :screen, :processor, :ram, :memory

  def initialize(model, screen_size, processor_freq, ram, memory, os)
    raise "Smartphone model required" if model.empty?
    raise "Screen size required" if screen_size.empty?
    raise "Processor Frequency required" if processor_freq.empty?
    raise "RAM required" if ram.empty?
    raise "Memory required" if memory.empty?
    raise "OS required" if os.empty?

    @model  = model
    @screen = Touchscreen.new(screen_size)
    @cpu    = CPU.new(processor_freq)
    @ram    = ram
    @memory = memory
    @os     = os
  end
end
{% endhighlight %}

Sure, this bloats the initializer, so we could extract the validation into a
separate method:

{% highlight ruby %}
class Smartphone
  attr_accessor :model, :screen, :processor, :ram, :memory

  def initialize(model, screen_size, processor_freq, ram, memory, os)
    validate_parameters!(model, screen_size, processor_freq, ram, memory, os)

    @model  = model
    @screen = Touchscreen.new(screen_size)
    @cpu    = CPU.new(processor_freq)
    @ram    = ram
    @memory = memory
    @os     = os
  end

  private
  def validate_parameters!(*p)
    model, screen_size, processor_freq, ram, memory, os = *p
    raise "Smartphone model required" if model.nil?
    raise "Screen size required" if screen_size.nil?
    raise "Processor Frequency required" if processor_freq.nil?
    raise "RAM required" if ram.nil?
    raise "Memory required" if memory.nil?
    raise "OS required" if os.nil?
  end
end
{% endhighlight %}

But, what would happen if we need to validate the CPU frequency? There isn't
really a CPU with a working frequency of 0 GHz, right? Or, maybe, validating
the Operating System?

{% highlight ruby %}
class Smartphone
  VALID_OS = [:ios, :android, :windows, :blackberry, :firefox, :tizen, :ubuntu]
  attr_accessor :model, :screen, :processor, :ram, :memory

  def initialize(model, screen_size, processor_freq, ram, memory, os)
    validate_parameters!(model, screen_size, processor_freq, ram, memory, os)
    validate_os!(os)

    @model  = model
    @screen = Touchscreen.new(screen_size)
    @cpu    = CPU.new(processor_freq)
    @ram    = ram
    @memory = memory
    @os     = os
  end

  private
  def validate_parameters!(*p)
    model, screen_size, processor_freq, ram, memory, os = *p
    raise "Smartphone model required" if model.nil?
    raise "Screen size required" if screen_size.nil?
    raise "Processor Frequency required" if processor_freq.nil?
    raise "RAM required" if ram.nil?
    raise "Memory required" if memory.nil?
    raise "OS required" if os.nil?
  end

  def validate_os!(os)
    unless VALID_OS.include?(os)
      raise "Invalid Operating System: #{os}"
    end
  end
end
{% endhighlight %}

This adds additional weight on the `Smartphone` class. One option to reduce the
complexity in the class would be to move this logic to a validator class. But,
since all of this logic is related to the configuration of a `Smartphone` object
on creation, moving the logic to the `SmarphoneBuilder` will ensure that **the
objects are always safely configurated**.

Let's refactor the `Smartphone` class and move the validation logic to the
`SmartphoneBuilder`:

{% highlight ruby %}
class Smartphone
  attr_accessor :model, :screen, :processor, :ram, :memory

  def initialize(model, screen_size, processor_freq, ram, memory, os)
    @model  = model
    @screen = Touchscreen.new(screen_size)
    @cpu    = CPU.new(processor_freq)
    @ram    = ram
    @memory = memory
    @os     = os
  end
end

{% endhighlight %}

First, we remove all of the validaton logic from the `Smartphone` class. The next
step is to find the perfect spot for the logic in the `SmartphoneBuilder` class:

{% highlight ruby %}
class SmartphoneBuilder
  VALID_OS = [:ios, :android, :windows, :blackberry, :firefox, :tizen, :ubuntu]
  attr_reader :smartphone

  def initialize
    @smartphone = Smartphone.new
  end

  def add_memory(gbs)
    validate_presence!("Memory", gbs)
    @smartphone.memory = gbs
  end

  def add_ram(gbs)
    validate_presence!("RAM", gbs)
    @smartphone.ram = gbs
  end

  def add_screen(screen_size)
    validate_presence!("Screen size", screen_size)
    @smartphone.screen = Touchscreen.new(screen_size)
  end

  def set_model(model)
    validate_presence!("Model", model)
    @smartphone.model = model
  end

  def add_processor(cpu_freq)
    validate_presence!("Processor Speed", cpu_freq)
    @smartphone.cpu = CPU.new(cpu_freq)
  end

  def set_os(os)
    validate_presence!("Operating System", os)
    validate_os!(os)
    @smartphone.os = os
  end

  def smartphone
    @smartphone
  end

  private
  def validate_presence!(attr_name, attr_value)
    raise "#{attr_name} is required." if attr_value.nil?
  end

  def validate_os!(os)
    unless VALID_OS.include?(os)
      raise "Invalid Operating System: #{os}"
    end
  end
end
{% endhighlight %}

As you can see, whenever we set any property on the builder, it validates the
configuration and `raise`s an error if the configuration is broken. In this
way, the Builder Pattern gives us the security that we will always get valid and
sane objects out of it. Let's say, we want to make sure that the phone memory
must be between 1GB and 64GBs, by adding additional validation:

{% highlight ruby %}
class SmartphoneBuilder
  VALID_OS = [:ios, :android, :windows, :blackberry, :firefox, :tizen, :ubuntu]
  attr_reader :smartphone

  def initialize
    @smartphone = Smartphone.new
  end

  def add_memory(gbs)
    validate_presence!("Memory", gbs)
    validate_memory!(gbs)
    @smartphone.memory = gbs
  end

  def add_ram(gbs)
    validate_presence!("RAM", gbs)
    @smartphone.ram = gbs
  end

  def add_screen(screen_size)
    validate_presence!("Screen size", screen_size)
    @smartphone.screen = Touchscreen.new(screen_size)
  end

  def set_model(model)
    validate_presence!("Model", model)
    @smartphone.model = model
  end

  def add_processor(cpu_freq)
    validate_presence!("Processor Speed", cpu_freq)
    @smartphone.cpu = CPU.new(cpu_freq)
  end

  def set_os(os)
    validate_presence!("Operating System", os)
    validate_os!(os)
    @smartphone.os = os
  end

  def smartphone
    @smartphone
  end

  private
  def validate_memory!(gigabytes)
    unless gigabytes.between?(1, 64)
      raise "The memory must be between 1GB and 64GBs"
    end
  end

  def validate_presence!(attr_name, attr_value)
    raise "#{attr_name} is required." if attr_value.nil?
  end

  def validate_os!(os)
    unless VALID_OS.include?(os)
      raise "Invalid Operating System: #{os}"
    end
  end
end
{% endhighlight %}

As you can see, adding more validation rules in the builder allows us to always
get valid objects, forever and ever.

## Outro

In this post we've covered, in my opinion, two of most popular cases where a
builder pattern can come in handy.

As a general rule, when you think of applying this pattern, look for places in
your code where confusion appears. Also, a good smell is a lot of configuration
overhead. In these places, you might find this pattern useful.

I am sure you might find other situations where you can apply this pattern. I
would love to learn of new places where this pattern can be used, so please
share your experience with me in the comments section below.

Thanks for reading!
