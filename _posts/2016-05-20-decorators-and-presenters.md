---
layout: post
title: "A bit about decorators and presenters"
tags: [ruby, decorators, presenters, odd]
excerpt: "Dive in to the decorator and presenter pattern and see what makes them simillar, but unique"
image: decorators-and-presenters.png
---

Object-oriented programming and design is (or, was?) a revolutionary way of
thinking and designing programs. It introduced classes, objects, inheritance,
polymorphism and many other ways to think about programming. As an addition, some
very smart folks identified some pitfalls and patterns that occur in
object-oriented programming and put them in books. That's how we got a list of
general code smells, design patterns and refactoring patterns that we can use in
our everyday work.

But, is it that simple? Unfortunately, some of the patterns look very similar on
the surface, and have subtle differences. Also, the craft lies in **applying**
these patterns, over learning them by heart. In the past, I've been always
confused between the decorator pattern and the presenter pattern. And if you are
(or used to be) confused by these, believe me - it's normal. Even the naming is
so similar, I would be surprised if someone new on the block is not confused.
Let's go over some examples and see what makes the similar, but different.

## Decorators

If you think about these patterns in a hierarchical way, the decorator pattern
would live on the top of the hierarchy. This means that, a presenter is *kind of*
subpattern of the decorator pattern. The decorator is a general purpose pattern,
whose role is to attach additional responsibilities to an object dynamically.
While "dynamically attach additional responsibilities" might sounds fancy, feel
free to think about it as "adding functionalities on the fly, when needed". Also,
the decorator pattern as a side-effect, provides a flexible alternative to
subclassing.

### Decorating over subclassing

Very often, when we have some hierarchy of classes, we use subclassing
(inheritance) to add some specific functionality to our subclass, while sharing
the inherited functionality from the superclass. In cases like this, the
decorator pattern gives us the flexibility to apply a decorator (or more
decorators) to a class, instead of creating a class hierarchy, to get additional
(or more specific) functionality on the class.

Also, the downside of subclassing is that it's static, which means it's applied
to a whole class. On the other hand, a decorator can be applied on runtime,
meaning it will decorate the object *when needed*.

### Example

Let's see a small example of what a decorator might look like. Ruby makes
defining decorators so flexible and easy, which makes it hard to choose which
way to do it.

Our base class will be a `CoffeeMachine`:

{% highlight ruby %}
class CoffeeMachine
  attr_reader :price

  def initialize(price)
    @price = price
  end
end
{% endhighlight %}

Now, some coffee machines have a milk steamer attached. To make a
`SteamedMilkCoffeeMachineDecorator` decorator, you can go couple of ways. The
first one would be to make it a PORO:

{% highlight ruby %}
class SteamedMilkCoffeeMachineDecorator
  def initialize(obj)
    @obj = obj
  end

  def price
    @obj.price + 300
  end

  def can_steam_milk?
    true
  end
end
{% endhighlight %}

Basically, when the object is decorated, it will override the `price` method
on the `CoffeeMachine` object, and it will also decorate a method
`can_steam_milk?`. The downside of using this approach is that we will need to
add some `method_missing` magic, because we want all of the methods that are not
present on the decorator to be delegated to the decorated object.

Or, we can use Ruby's
[SimpleDelegator](http://ruby-doc.org/stdlib-2.1.0/libdoc/delegate/rdoc/SimpleDelegator.html)
:

{% highlight ruby %}
class SteamedMilkCoffeeMachineDecorator < SimpleDelegator
  def price
    super + 300
  end

  def can_steam_milk?
    true
  end
end
{% endhighlight %}

This decorator will have the same functionality as the PORO decorator above, with
the addition of delegating the methods, that are unknown to the decorator, to
the wrapped object.

Using the decorator is plain simple:

{% highlight ruby %}
coffee_machine = CoffeeMachine.new(500)
with_steamer = SteamedMilkCoffeeMachineDecorator.new(coffee_machine)

coffee_machine.price
#=> 500

with_steamer.can_steam_milk?
#=> true

with_steamer.price
#=> 800
{% endhighlight %}

If you would like to see a nice summary of decorator implementations, I would
recommend you read
[Evaluating Alternative Decorator Implementations In Ruby](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in)
by Dan Croak.

## The Presenter pattern

Now that we have a good grasp of the decorator pattern, let's see what it is,
and where the presenter pattern shines. As we mentioned in the beginning of this
article, the presenter is a "subpattern" of the decorator. The main difference
between them is how close they live to the view. Presenters live very close to
the view layer, while decorators being very broad, can live near the business
logic of your application.

Within a Rails application, presenters can be seen in various shapes. Most often,
presenters are used to keep logic out of the views:

{% highlight ruby %}
class Apartment
  attr_reader :area

  def initialize(sq_meters: sq_meters)
    @area = sq_meters
  end
end

class ApartmentPresenter < Decorator
  decorates :apartment

  def size
    if area < 30
      "Small"
    elsif area < 50
      "Cozy"
    elsif area < 80
      "Big"
    else
      "Huge"
    end
  end
end
{% endhighlight %}

As you can tell, this example uses the very popular
[draper](https://github.com/drapergem/draper) gem that enables easy creation of
presenters. We can use our newly created presenter as:

{% highlight ruby %}
apartment = ApartmentPresenter.new(Apartment.new(sq_meters: 75))
apartment.area #=> 75
apartment.size #=> "Big"
{% endhighlight %}

With presenters like this, you can always simplify your views. Whatever the type
of logic that you usually would put into helper methods, you can use presenters
for it. Presenters are more object-oriented way to achieve the same goal.

## Wrapping up

If you think just a bit more broadly about decorators, you can see that basically
decorators are the open/closed principle (the O, from the SOLID principles) put
into practice. The open/closed principle states that a class should be open
for extension, but closed for modification, which if applied in our context,
easily paints the picture of how decorators extend classes.

As it usually goes, when something is easy to create, abusing it even easier.
Gems like Draper allow us to create decorators without any hassle, so it's pretty
easy to create fat decorators or to use them in weird ways. Also, another thing
to keep in mind is that although decorators help with applying SOLID principles
to your code, the decorators themselves need to oblige to these principle at the
same time.

Nevertheless, learning about the decorator pattern is a very useful addition to
your design patterns toolbelt. Although at time using it in the proper manner can
require some additional thinking (and maybe even prototyping), it can improve the
flexibility of your design.

## Additional reading

These are some links that I would recommend you if you would like to learn more
about the decorator pattern:

- [Exhibit vs Presenter](http://mikepackdev.com/blog_posts/31-exhibit-vs-presenter) by Mike Pack
- [Decorator Design Pattern](https://sourcemaking.com/design_patterns/decorator) at SourceMaking
- [Rails Anti-Pattern: Fat Decorator](http://craftingruby.com/posts/2015/12/09/rails-antipattern-fat-decorator.html) by Jeroen Weeink
- [Draper](https://github.com/drapergem/draper) gem
- `SimpleDelegator` [documentation](http://ruby-doc.org/stdlib-2.1.0/libdoc/delegate/rdoc/SimpleDelegator.html)
