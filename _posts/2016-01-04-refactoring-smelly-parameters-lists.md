---
layout: post
title: "Refactoring in Ruby: Smelly Parameters Lists"
tags: [refactor, ruby, parameters, smells]
---

Ruby is a really clear and expressive language, but we developers sure know how
to make a mess. Even when you think your classes are nicely written and tested,
things can still get out of hand. I am pretty sure you've heard one (or more)
of your more experienced colleagues/mentors tell you that "something is smelly"
in the code. Well, in this article we will cover one of the simplest code smells
- long parameters lists in your method signatures.

## Code Smells

First, what are code smells? According to the
[Wikipedia article](https://en.wikipedia.org/wiki/Code_smell){:target="_blank"}
on Code smell:

> Code smell, also known as bad smell, in computer programming code, refers to
> any symptom in the source code of a program that possibly indicates a deeper
> problem.

Another very imporant point is this:

> Code smells are usually not bugs—they are not technically incorrect and do
> not currently prevent the program from functioning. Instead, they indicate
> weaknesses in design that may be slowing down development or increasing the
> risk of bugs or failures in the future. Bad code smells are an important
> reason for technical debt.

These couple of sentences explain what code smells are very nicely. If you would
like to know more, I encourage you to head over to the Wikipedia article.

Now, back to our code smell - long parameters list.

## Long Parameters Lists

This code smell is quite easy to detect - just look for methods with 3, 4 or
more parameters. Now, while to some of you this might be okay, let's see why
this is an actual code smell.

{% highlight ruby %}
class Person
  def initialize(first_name, last_name, age, sex, height, weight)
    @first_name = first_name
    @last_name = last_name
    @age       = age
    @sex       = sex
    @height    = height
    @weight    = weight
  end
end
{% endhighlight %}

Just creating a new object of the `Person` class can lead to bugs. For example,
it's really easy to mix up the height and weight, also the first and last name.
Then, just reading the signature of the initializer is super hard. And even if
this long of an initializer signature can be acceptable to you, imagine doing
this for any other method that you do not invoke that often.

Take this for example:

{% highlight ruby %}
class Invitation
  def self.deliver!(first_name, last_name, email, date, time, venue, dress_code)
    Invitation.create(to: "#{first_name} #{last_name}", email: email)
    UserMailer.invitation(first_name, last_name, date, time, venue, dress_code)
  end
end
{% endhighlight %}

I hope that this paints the picture well that long paratemeters lists can lead
to bugs and confusion. So, how do we refactor them?

## Replace Parameter with Method Call

One refactoring pattern that could help us here is the Replace Parameter with
Method Call pattern. Let's take this as an example:

{% highlight ruby %}
class Discount
  def initialize(store, price, quantity)
    @store = store
    @price = price
    @quantity = quantity
  end

  def effective_cost
    base_price = @quantity * @price
    discount = @store.seasonal_discount
    apply(base_price, discount)
  end

  def apply(base_price, discount)
    base_price - discount
  end
end
{% endhighlight %}

As you can see, we use the `Discount#effective_cost` method to calculate a
discounted price. Now, the problem lies where we need to pass in the discount as
parameter to the `Discount#apply` method. Instead of doing that, we can reduce
the parameters list of the `apply` method to one.

This can be achieved by substituting the parameter with a method call and
pushing the logic of retrieving the discount down to the `apply` method:

{% highlight ruby %}
class Discount
  def initialize(store, price, quantity)
    @store = store
    @price = price
    @quantity = quantity
  end

  def effective_cost
    base_price = @quantity * @price
    calculate_effective_cost(base_price)
  end

  def apply(base_price)
    discount = @store.seasonal_discount
    base_price - discount
  end
end
{% endhighlight %}

As you can see, by refactoring the parameter into to a method call, we reduced
the number of parameters and made the method signatures much cleaner.

## Preserving a Whole Object

Now, using the same example, let's take a look at another refactoring pattern -
the Preserve Whole Object pattern. The example that we will use is the
refactored version of the code from the last section:

{% highlight ruby %}
class Discount
  def initialize(store, price, quantity)
    @store = store
    @price = price
    @quantity = quantity
  end

  def effective_cost
    base_price = @quantity * @price
    calculate_effective_cost(base_price)
  end

  def apply(base_price)
    discount = @store.seasonal_discount
    base_price - discount
  end
end
{% endhighlight %}

Preserve Whole Object states that instead of passing raw data as parameters, one
can send the same data encapsulated in an object. Let's see how this pattern can
improve our code:

First, instead of passing in the price and the quantity as separate data, we can
introduce a new object to our program - a `Order`. The `Order` will hold the
price of the product and the quantity that was ordered.

{% highlight ruby %}
class Order
  attr_reader :price, :quantity
  def initialize(product, quantity)
    @price    = product.price
    @quantity = quantity
  end
end
{% endhighlight %}

Then, in the initializer of our `Discount` class, instead of passing in the raw
data we will pass an `Order` object as a parameter:

{% highlight ruby %}
class Discount
  def initialize(store, order)
    @store = store
    @order = order
  end

  def effective_cost
    base_price = @order.quantity * @order.price
    calculate_effective_cost(base_price)
  end

  def calculate_effective_cost(base_price)
    discount = @store.seasonal_discount
    base_price - discount
  end
end
{% endhighlight %}

As you can see, by taking this step, the list of parameters in the initializer
of the `Discount` class fell down from three to two.

## Introduce Parameter Object pattern

Another way of getting rid of long parameters lists is the Introduce Parameter
Object pattern. The pattern basically states that when a certain list of
parameters has a **solid logical link** between them, it is a good practice to
wrap them in a data structure/object. This will make the code more testable and
will improve it's readibility.

Personally, I would enforce using this pattern if the parameters are being
repeated in multiple places/classes/methods.

For this section, we will change the example. We will use a `Invoice` class
which has an issue date and a due date:

{% highlight ruby %}
class Invoice
  def self.deliver(issue_date, due_date)
    # Deliver the invoice here somehow...
  end

  def self.store(issue_date, due_date)
    # Store the invoice here somehow...
  end
end
{% endhighlight %}

Now, since we have a logical link between the parameters here (and duplication
in the same time), we can extract the `issue_date` and `due_date` into a
`DateRange` object:

{% highlight ruby %}
class DateRange
  def initialize(start_date, end_date)
    @start_date = start_date
    @end_date   = end_date
  end

  def started?
    Date.today > @start_date
  end

  def ended?
    Date.today > @end_date
  end
end
{% endhighlight %}

As you can see, we have made the `DateRange` class as a general purpose class
for any other date ranges. Now, instead of passing the dates separately as
parameters, we can pass in a DateRange object in the `Invoice` class:

{% highlight ruby %}
class Invoice
  def self.deliver(date_range)
    if date_range.ended?
      # Deliver with a warning notice
    else
      # Just deliver the invoice
    end
  end

  def self.store(date_range)
    if date_range.started?
      # Store the invoice here somehow...
    end
  end
end
{% endhighlight %}

As you can see, this allows us to easily modify the behaviour of the methods in
the `Invoice` class, adding points to readibility and testability of the
methods. Also, the `DateRange` class is dead-easy to test at this point.

## Taking a step back

The three patterns we covered in this article are a nice way to remove the code
smell around long parameters lists. I guess that some of them are just an
intuitive way to remove the code smell to some. If that is so - great, you do
not have to see them as patterns. If not - not a big deal, always look for
logical links in the parameters and try to encapsulate the data in some way.
With a bit of practice, you can easily apply these patterns on your own.

Hope you enjoyed this refactoring article. If you would like to learn more about
refactoring, check out [the article](/tdd-extract-class){:target="_blank"} on
the Extract Class refactoring pattern.

