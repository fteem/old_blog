---
layout: post
title: "Exploring new methods in Ruby 2.2.0"
tags: [angularjs, google, javascript]
---

For those late to the Ruby 2.2.0 party like me, aside from the changes (and updates) the core team made under the hood for this version, they introduced couple of new methods to the _Enumerable_ module and to the_ Method, Float, File_ and _String_ classes. Lets take a look at these methods and explore how we can use them in our everyday jobs. Just a heads up, make sure you **use Ruby 2.2.0** when working on the examples.

### Enumerable's #slice_after

**Enumerable#slice_after** is used to chunk an array. When chunks are created, it creates an enumerator for the chunked elements. The end of every chunk is defined by a pattern or a block. This means that, if the pattern returns true for an element, the element that matches the pattern is the end of the chunk. This applies for the block as well - if the block returns true for an element - that element is the end of the chunk. How can we use this? Say you have some an array of page numbers and you want to group them by three. Also, the group should show the first page and last page of the group, with a dash in between.

{% highlight ruby %}
pages = Array(1..12)
# => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]

enum = pages.slice_after{|n| n % 3 == 0 }

enum.to_a
# => [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]

enum.map {|chunk| "#{chunk[0]} - #{chunk[2]}" }
# => ["1 - 3", "4 - 6", "7 - 9", "10 - 12"]
{% endhighlight %}

You can see that we use slice_after to slice the array after every third number. This returns an enumerator, that we can use to loop over it and do something with the chunked elements. In this case, we concatenate the first and the last element with a dash in between them, and we compose a new array of the grouped page numbers by using **Enumerable#map**. You can read more about the slice_after method in the [official Ruby docs](http://ruby-doc.org/core-2.2.0/Enumerable.html#method-i-slice_after).

### Enumerable's #slice_when

**slice_when** works very similarly to **slice_after**. The difference is that it only accepts a block as an argument, and it will create a chunk when that block returns true. Also, the block accepts two arguments, representing two adjacent elements of the array. How can we use this? Lets say we have an array of numbers and we want to detect all decreasing subsequences in the array.

{% highlight ruby %}
# Generates an array of 20 random numbers from 1 to 10K
arr = (1..10000).to_a.sample 20
# => [9000, 4068, 3339, 234, 2861, 2436, 1853, 397, 701, 7072, 6306, 6976, 5736, 4942, 7332, 4941, 6171, 2178, 248, 8473]

arr.slice_when {|left, right| left < right }.to_a
# => [[9000, 4068, 3339, 234], [2861, 2436, 1853, 397], [701], [7072, 6306], [6976, 5736, 4942], [7332, 4941], [6171, 2178, 248], [8473]]
{% endhighlight %}

As you can see, the **slice_when** method block takes two arguments, the number "on the left" and the number "on the right". I find it easier to think about it that way, but I think it is more correct to use _previous_ and _next_ when naming the block arguments. In the example, the code in the block just looks for a pair of adjacent elements in the array where the first one is smaller than the second one. When the condition returns true a chunk is created. You can read more about this method in the [official Ruby docs.](http://ruby-doc.org/core-2.2.0/Enumerable.html#method-i-slice_when)

### Float's #next_float and #prev_float

These are pretty much self explanatory. When you call any of these methods on a floating point number, you get the next/previous in line.

Keep in mind that if you call 

{% highlight ruby %}
Float::MAX.next_float
{% endhighlight %}
you will get back *Infinity*. For those that haven't seen *Float::MAX* before,
it's the largest floating point number that Ruby can interpret (it's value is 1.7976931348623157e+308).
Also, if you call

{% highlight ruby %}
(-Float::MAX).prev_float
{% endhighlight %}
you will get back *-Infinity*. You can read more about these methods in the official docs for [next_float](http://ruby-doc.org/core-2.2.0/Float.html#method-i-next_float) and [prev_float](http://ruby-doc.org/core-2.2.0/Float.html#method-i-prev_float).

### File's .birthtime and #birthtime

Also, very self explanatory. It returns the time and date when the file was created.

{% highlight ruby %}
# You can use the class method with the path of the file
File.birthtime("/Applications/TextEdit.app")
# => 2014-09-10 00:05:30 +0200

# Or, open the file...
f = File.open("/Applications/TextEdit.app")
# .. and call #birthtime.
f.birthtime
# => 2014-09-10 00:05:30 +0200

# You can also use the class method with a File object.
File.birthtime(f)
# => 2014-09-10 00:05:30 +0200
{% endhighlight %}

You can read more about this in the [official docs](http://ruby-doc.org/core-2.2.0/File.html#method-c-birthtime) for the File class. **Important**: this does not work on GNU/Linux, because (at the moment of writing this) Linux has no API where Ruby would read that data from. Also, it seems that **only Ext4** filesystem is keeping this data, but as said, it doesn't expose it. You can read more about this on [this LKML mailing list thread](https://lkml.org/lkml/2010/7/22/249).

### String's #unicode_normalize, #unicode_normalize! & #unicode_normalized?

**#unicode_normalize **normalizes the string. The predicate checks if the string is  normalized.

{% highlight ruby %}
# The Unicode code for the umlaut sign is \u0308
bad = "making u with an umlaut in Unicode is done with: u\u0308"
# => "making u with an umlaut in Unicode is done with: ü"

bad.unicode_normalized?
# => false

bad.unicode_normalize!
# => "making u with an umlaut in Unicode is done with: ü"

bad.unicode_normalized?
# => true
{% endhighlight %}

### Method#super_method

This method returns a Method of superclass, which would be called if _super_ is used.

{% highlight ruby %}
class Parent
  def name
    puts "I'm the parent!"
  end
end

class Child < Parent
  def name
    puts "I'm the child!"
  end
end

parent = Parent.new
parent.name
# I'm the parent!

child = Child.new
child.name
# I'm the child!

# We need to access the Method object, not the result of method...
# So, this will not work because the method returns nil, not itself:

child.name.super_method
# NoMethodError: undefined method `super_method' for nil:NilClass

child.name.class
# => NilClass

# But if we access the Method object...
child.public_method(:name)
# => #<Method: Child#name>
# ... we can call the super method of the #name method

child.public_method(:name).super_method
# => #<Method: Parent#name>
child.public_method(:name).super_method.call
# I'm the parent!
{% endhighlight %}

### Outro

These are some nice methods that can make our lives easier. I hope this blog post helped you understand (and maybe discover) these new methods that were added to Ruby 2.2.0. What do you think about these new methods? Where can you put them in use? Please, feel free to drop a comment - I would love to read your opinion on this topic.
