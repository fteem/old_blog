---
layout: post
title: "SOLID Principles in Ruby"
---

Regardless of your knowledge level, as a programmer you love to write awesome code. It's what we do. We like it and we do it every single day.
But, we all know that writing awesome code is not easy at all. So, how can we improve the code we produce every day?

An awareness (or a reminder!) of SOLID principles is beneficial here. SOLID is a group of five principles that when *applied correctly* can help us produce better code.

#### So, what are the SOLID principles?

SOLID is a mnemonic acronym coined by [Uncle Bob](http://en.wikipedia.org/wiki/Robert_C._Martin) back in the early 2000s. It represents a group of five principles:

- **S**ingle Responsibility Principle
- **O**pen/Closed Principle
- **L**iskov Substitution Principle
- **I**nterface Segregation Principle
- **D**ependency Inversion Principle

Sounds good? Great. Let's take a look at each of these principles.
<hr/>
#### Single Respnosibility Principle

In my opinion, this is the easiest principle to understand. What SRP says is:

> Every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class.

What does this mean? Basically, every class in your app **must** have a single responsibility. Easy as that. The best way to detect if your class obeys this principle is to answer this question:

> **What does this class do?**

If your answer contains the word **AND**, then your class does not obey the SRP.

Lets see a quick example. There's the Student class and every student has grades for different terms.


{% highlight ruby %}

class Student
  attr_accessor :first_term_home_work, :first_term_test,
    :first_term_paper
  attr_accessor :second_term_home_work, :second_term_test,
    :second_term_paper

  def first_term_grade
    (first_term_home_work + first_term_test + first_term_paper) / 3
  end

  def second_term_grade
    (second_term_home_work + second_term_test + second_term_paper) / 3
  end

end
{% endhighlight %}

Some of you may already be thinking "that is wrong sir!", some of you may not. Regardless of that, yes, this **does not** obey the SRP. The reason is that the class Student contains the logic that calculates the average grade for each term. The responsibility of ```Student``` is to hold info/logic about the student, not the grades. The logic that calculates the grades should be part of a class ```Grade```, not ```Student```.

Let's refactor the code.

{% highlight ruby %}
class Student
  def initialize
    @terms = [
      Grade.new(:first),
      Grade.new(:second)
      ]
  end

  def first_term_grade
    term(:first).grade
  end

  def second_term_grade
    term(:second).grade
  end

  private

  def term reference
    @terms.find {|term| term.name == reference}
  end
end

class Grade
  attr_reader :name, :home_work, :test, :paper

  def initialize(name)
    @name      = name
    @home_work = 0
    @test      = 0
    @paper     = 0
  end

  def grade
    (home_work + test + paper) / 3
  end
end
{% endhighlight %}

You can see that now ```Grade``` holds the logic for calculation of the grade and Student only stores the grades into a collection. Now this complies to the SRP, because, every class has it's own responsibility.

<hr/>
#### Open/closed principle

The Open/closed principle (OCP) is a principle whose definition is:

> One software entity (class/module) must be open for extension but closed for modification.

What does this mean? Once a class implements the current scope of requirements, the implementation should not need to change in order to fulfil future requirements.

It doesn't make sense? Let's take a look at a quick example.

{% highlight ruby %}
class MyLogger
  def initialize
    @format_string = "%s: %s\n"
  end

  def log(msg)
  	STDOUT.write @format_string % [Time.now, msg]
  end
end
{% endhighlight %}
Simple logger class right? It has a format string and sends the current time and the message to STDOUT. Cool, simple enough. Lets test it:

{% highlight bash %}
irb> MyLogger.new.log('test!')
=> 2014-04-25 16:16:32 +0200: test!
{% endhighlight %}

Awesome. But, what would happen if someone in the future needs the logger to prepend the string "[LOG]" to the log message, so the output would look like:

{% highlight bash %}
[LOG] 2014-04-25 16:16:32 +0200: MyLogger calling!
{% endhighlight %}

For example, a programmer that does not know about the OCP can possibly do this change:

{% highlight ruby %}
class MyLogger
  def initialize
    @format_string = "[LOG] %s: %s\n"
  end
end
{% endhighlight %}

And the output of the new MyLogger class would be:

{% highlight bash %}
irb> MyLogger.new.log('test!')
=> [LOG] 2014-04-25 16:16:32 +0200: test!
{% endhighlight %}

Everything looks good, right? But, wait a second? Does it?

Think about this - if this was a core class of an app, the change we introduced to the ```format_string``` would break the functionality of that classes that rely on the ```MyLogger``` class. There's the possibility that a whole world out there relies on the former funcionality of the class, but now, that we changed it, a lot of things can break. This is a violation of the OCP and it is **bad**!

So, what is the good way to do it? Inheritance! Or object composition!

Let's see an example that uses inheritance:

{% highlight ruby %}
class NewCoolLogger < MyLogger
  def initialize
    @format_string = "[LOG] %s: %s\n"
  end
end
{% endhighlight %}

{% highlight bash %}
irb> NewCoolLogger.new.log('test!')
=> [LOG] 2014-04-25 16:16:32 +0200: test!
{% endhighlight %}

Nice, works as expected! What about the functionality of ```MyLogger```?

{% highlight bash %}
irb> MyLogger.new.log('test!')
=> 2014-04-25 16:16:32 +0200: test!
{% endhighlight %}

Great! So, what did we just do? We extended the ```MyLogger``` class and created a brand new class called ```NewCoolLogger``` that extends the former class. Now the code that relies on the functionality of the old logger will not break due to the changes we introduced. The old logger will work just like it did before and the new one will provide the new functionality that the programmer wanted.

Also, I mentioned *object composition*. Take this refactor in cosideration:

{% highlight ruby %}
class MyLogger
  def log(msg, formatter: MyLogFormatter.new)
    STDOUT.write formatter.format(msg)
  end
end
{% endhighlight %}

You can notice that the log method receives an optional parameter called ```formatter```.  The format of the log string is responsibility of the ```MyLogFormatter``` class not the logger class itself. This is good because now ```MyLogger#log``` can accept different formatter classes that will set the format of the log message. For example, you can create ```ErrorLogFormatter``` that will prepend ```[ERROR]``` to the log message but ```MyLogger``` will not care because all it needs is a string that it will send to STDOUT.

<hr/>

#### Liskov substitution principle

[Barbara Liskov](http://en.wikipedia.org/wiki/Barbara_Liskov) defined the principle within these lines:

> If S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program (correctness, task performed, etc.).

Honestly, I found this definition pretty hard to understand. So, after some thinking, this is what it boils down to:

There is a class ```Bird```. And there are two objects, ```obj1``` and ```obj2```. The class of ```obj1``` is ```Duck``` which is a child-class of ```Bird```. Let's say we discover that ```obj2```'s class is ```Pigeon```, which is also a child-class of ```Bird```. Liskov substitution principle states that in this situation, when ```obj2``` has a type of ```Bird``` sub-class and ```obj1``` which is of class ```Duck``` which is also a sub-type of ```Bird```, I should be able to treat ```obj1``` and ```obj2``` in the same way - as ```Bird```s.

Still confusing? Take a look at the example below.

{% highlight ruby %}
class Person
  def greet
    puts "Hey there!"
  end
end

class Student < Person
  def years_old(age)
    return "I'm #{age} years old."
  end
end

person = Person.new
student = Student.new

# What LSP says is if I know the interface of person, I need to be able to guess the interface of student because
# the Student class is a subtype of the Person class.
student.greet
# returns "Hey there!"
{% endhighlight %}

Hope that explained LSP.
<hr/>

#### Interface segregation principle

The interface-segregation principle (ISP) states that:
> No client should be forced to depend on methods it does not use.

Simple as that. Lets see some code examples and explain them.

{% highlight ruby %}
class Computer
  def turn_on
    # turns on the computer
  end

  def type
  	# type on the keyboard
  end

  def change_hard_drive
    # opens the computer body
    # and changes the hard drive
  end
end

class Programmer
  def use_computer
    @computer.turn_on
    @computer.type
  end
end

class Technician
  def fix_computer
    @computer.change_hard_drive
  end
end
{% endhighlight %}

In this example, there are ```Computer```, ```Programmer``` and ```Technician``` classes. Both, ```Programmer``` and ```Technician``` use the ```Computer``` in a different way. The programmer uses the computer for typing, but the technician knows how to change the computer hard drive. What Interface Segregation Principle (ISP) enforces is that one class should not depend on methods it does not use.
In our case, ```Programmer``` is unnecessarily coupled to the ```Computer#change_hard_drive``` method because it does not use it, but the state changes that this method enforces can affect the ```Programmer```. Let's refactor the code to obey the LSP.

{% highlight ruby %}
class Computer
  def turn_on
  end

  def type
  end
end

class ComputerInternals
  def change_hard_drive
  end
end

class Programmer
  def use_computer
    @computer.turn_on
    @computer.type
  end
end

class Technician
  def fix_computer
    @computer_internals.change_hard_drive
  end
end
{% endhighlight %}

After this refactor the ```Technician``` uses a different object from the type ```ComputerInternals``` which is isolated from the state of the ```Computer```. The state of the ```Computer``` object can be influenced by the ```Programmer``` but the changes wont affect the ```Technician``` in any way.

<hr/>
#### Dependency inversion principle

Dependency inversion principle refers to a specific form of decoupling software modules. It's definition has two parts:

> 1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
2. Abstractions should not depend upon details. Details should depend upon abstractions.

I know that this might be a bit confusing. But, before we jump to a example I want to make sure that you **must not mix** *Dependecy Inversion Principle* with *Dependency Injection*. The later is a technique (or pattern) and the former is the principle.

Having that said, lets see the example:

{% highlight ruby %}
class Report
  def initialize
    @body = "whatever"
  end

  def print
    XmlFormatter.new.generate @body
  end
end

class XmlFormatter
  def generate(body)
    # convert the body argument into XML
  end
end
{% endhighlight %}

The ```Report``` class is used to generate an XML report. In it's initializer we setup the report and its body. The ```print``` method uses the ```XmlFormatter``` class to convert the body of the report to XML. Easy as that.

Let's think a bit about this class. Look at it's name - ```Report```. It's a generic name and it tells us that it will return a report of some kind, but, it doesnt say much about it's format. In fact, in our example, we can easily rename our class to ```XmlReport``` since we know the implementation details. But insead of making it very specific, let's think about abstracting this code.

Right now, our class is dependant on the ```XmlFormatter``` class and it's interface i.e. ```generate```. ```Report``` right now is dependent on a detail, not on abstraction. It knows that there must be a class ```XmlFormatter``` so it can work. Also, another question - what would happen if we wanted an CSV report? Or a JSON report? We'd have to have multiple methods like ```print_xml```, ```print_csv``` or ```print_json```. This means that our class right now is very tied to the details, it knows about the formatter class type instead of knowing just how to use it (abstraction).

Let's refactor it:

{% highlight ruby %}
class Report
  def initialize
    @body = "whatever"
  end

  def print(formatter)
    formatter.generate @body
  end
end

class XmlFormatter
  def generate(body)
    # convert the body argument into XML
  end
end
{% endhighlight %}

Look at the ```print``` method now. It knows that it needs a formatter, but it only cares about it's interface. To be more specific, it only cares that that formatter has a method called ```generate```. How is this better? Well, if we wanted CSV reports, all we would need is to add the following class:

{% highlight ruby %}
class CSVFormatter
  def generate(body)
    # convert the body argument into CSV
  end
end
{% endhighlight %}

The ```Report#print``` method would accept a ```CSVFormatter``` object as a parameter which would convert the report body into a CSV string.
<hr/>

That pretty much sums up all of the five SOLID principles. All of our refactoring examples are very basic and we just scratched the surface. I'm sure that in your carreer as a programmer you'd come on to much more complex problems. But, be assured that having **SOLID** foundations can definitely help you to write better code that is easier to maintain.


##### This post was originaly published by me on [Siyelo's blog](http://blog.siyelo.com/solid-principles-in-ruby/). Thanks to them for helping me write it!
