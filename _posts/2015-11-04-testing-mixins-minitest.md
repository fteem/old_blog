---
layout: post
title: "Testing Ruby Mixins with Minitest in isolation"
tags: [ruby, minitest, mixins, isolation]
---

Mixins in Ruby are a very powerful feature. But knowing how to test them sometimes 
is not so obvious, especially to beginners. I think that this comes from mixins' 
nature - they get mixed into other classes. So, if you think that there is a lot 
to testing mixins and you easily get overwhelmed by it - take it easy, it's not 
**that** hard. 

Let's see how easy it is to test mixins, with some help from the lovely Minitest.

## Mixins

When writing Ruby, we usually write two types of mixins. The first one is a coupled
mixin, and the other, well.. uncoupled.

### Uncoupled mixins

Uncoupled mixins are the ones whose methods **do not** depend on the implementation 
of the class where they will get mixed in. 

Quick example:

{% highlight ruby %}
module Speedable
  def speed
    "This car runs super fast!"
  end
end

class PetrolCar
  include Speedable
  def fuel
    "Petrol"
  end
end

class DieselCar
  include Speedable
  def fuel
    "Diesel"
  end
end
{% endhighlight %}

In `irb`:

{% highlight ruby %}
>> p = PetrolCar.new
=> #<PetrolCar:0x007fc332cc4be0>
>> p.speed
=> "This car runs super fast!"
>> p.fuel
=> "Petrol"

>> d = DieselCar.new
=> #<DieselCar:0x007fc332cae2f0>
>> d.speed
=> "This car runs super fast!"
>> d.fuel
=> "Diesel"
{% endhighlight %}

As you can see, the `speed` method in the `Speedable` mixins does not depend on 
any other methods. In other words, it's self-contained.

#### Testing uncoupled mixins

When it comes to testing uncoupled mixins, it's quite trivial. There are two main
strategies that you can use, by extending the singleton class of an object or by 
using a **dummy class**.

Let's see the first one.

{% highlight ruby %}
class FastCarTest < Minitest::Test 
  def setup
    @test_obj = Object.new
    @test_obj.extend(Speedable)
  end
  
  def test_speed_reported
    assert_equal "This car runs super fast!", @test_obj.speed
  end
end
{% endhighlight %}

As you can see, we instantiate an object of the `Object` class which is just
an empty, ordinary object that doesn't do anything. Then, we extend the object 
singleton class with the `Speedable` module which will mix the `speed` method in.
Then, in the test, we assert that the method will return the expected output.

The second strategy is the "dummy class" strategy:

{% highlight ruby %}
class DummyTestClass
  include Speedable
end

class FastCarTest < Minitest::Test 
  def test_speed_reported
    dummy = DummyTestClass.new
    assert_equal "This car runs super fast!", dummy.speed
  end
end
{% endhighlight %}

As you can see, we create just a dummy class, specific only for this test file. 
Since the `FastCar` mixin is mixed in, the `DummyTestClass` will have the
`speed` method as an instance method. Then, in the test, we just create a new
object from the dummy class and assert on the `dummy.speed` method.

### Coupled mixins

Coupled mixins are the ones whose methods depend on the implementation of the class
where they will be mixed in. Or, the oposite of uncoupled mixins. 

Let me show you a quick example:

{% highlight ruby %}
module Reportable
  def report
    "This car runs on #{fuel}."
  end
end

class PetrolCar
  include Reportable
  def fuel
    "petrol"
  end
end

class DieselCar
  include Reportable
  def fuel
    "diesel"
  end
end
{% endhighlight %}

Now, if we try our classes in `irb`:

{% highlight ruby %}
>> pcar = PetrolCar.new
=> #<PetrolCar:0x007fda3403bba8>
>> pcar.report
=> "This car runs on petrol."

>> dcar = DieselCar.new
=> #<DieselCar:0x007fda3318a320>
>> dcar.report
=> "This car runs on diesel."
{% endhighlight %}

You see, the implementation of the `Reportable#report` method relies on 
(or, is coupled to) the implementation of the `DieselCar` and `PetrolCar` 
classes. If we mixed in the `Reportable` mixin in a class that does not have the 
`fuel` method implemented, we would get an error when calling the `report` method.

#### Testing coupled mixins

When it comes to coupled mixins, testing can get just a tad bit harder. Again, the same two
strategies.
 
{% highlight ruby %}
class ReportableTest < Minitest::Test 
  def setup
    @test_obj = Object.new
    @test_obj.extend(Reportable)

    class << @test_obj
      def fuel
        "diesel"
      end
    end
  end
  
  def test_speed_reported
    assert_equal "This car runs on diesel.", @test_obj.report
  end
end
{% endhighlight %}

Now, you can see, it gets a bit hairy when we open the singleton class of the `@test_obj`
and we add the `fuel` method, so our coupled mixin can work. But, otherwise,
it's quite straight forward.

The approach that I prefer is using the dummy class because it's much more explicit.

{% highlight ruby %}
class DummyCar
  include Reportable

  def fuel
    "gasoline"
  end
end

class ReportableTest < Minitest::Test 
  def test_fuel_reported
    dummy = DummyCar.new
    assert_equal "This car runs on gasoline.", dummy.report
  end
end
{% endhighlight %}

Now, this is cleaner. We create a `DummyCar` class where we mix the `Reportable`
mixin and we define the `fuel` method. Then, in the test, we just create a
`DummyCar` object and we assert for the value of the `report` method. Remember,
we are doing this only because we want to test the mixin. If we were to test any of 
the classes, there would be no point in doing this.


As you can see, it's quite simple to test these mixins with plain-old Ruby. I guess 
it's worth mentioning that these examples are very simple and you might run into 
more complex mixins that need testing in the future. But, when testing mixins, 
these strategies will still work. Just don't get owerwhelmed and take it step-by-step.
