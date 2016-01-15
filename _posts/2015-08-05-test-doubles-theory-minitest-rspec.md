---
layout: post
title: "Test Doubles: in theory, in Minitest and in RSpec"
tags: [ruby, testing, tdd, double, mock, stub, fake, dummy]
---

Those of us that do **T**est **D**riven **D**evelopment have heard about doubles, 
mocks, stubs, fakes and spies multiple times. Unfortunately there is **a ton** of 
confusion about all these words and their meaning. Let's see what each an every 
one of these really mean, where we should use them and how the popular testing 
frameworks for Ruby implement these.

## Test Doubles

So, first things first. One of the biggest misconceptions is that doubles are 
types of objects that are used in testing. 

Dummies, mocks, stubs, fakes and spies **are test doubles**. Test double is the 
**category** of these test objects. I think that the confusion around this concept
arises from the naming that the popular testing frameworks use for these objects.

So remember - dummies, mocks, stubs, fakes and spies **are test doubles**.
Also, another thing to keep in mind is the word **mock**. The words mock and 
doubles were used interchangeably in the past and their definitions got skewed.

## Dummy

Now, that we got this right, what is a dummy?

The simplest type of test double is test dummy. Basically, the dummy is dumb, 
hence the naming. This means that the object is just a placeholder for another 
object, but it's interfaces (or methods) do not return anything.

Confusing? Let's see an example.

We have this class ```Blog``` that has a ```publish!``` method.

{% highlight ruby %}
class Post
  attr_reader :published

  def publish!
    @published = true
  end
end

class Blog
  attr_reader :published_posts

  def initialize author
    @author = author
    @published_posts = []
  end

  def publish! post
    published_posts << post
    post.publish!
  end

  # Moar methods...
end
{% endhighlight %}

Now, we need to test the ```publish!``` method. How do we go about this? We obviously 
need to create a ```Post``` object first, then create a ```Blog``` object and pass 
the author as an argument to the initializer. Afterwards, we need to assert that 
the ```publish!``` method will add the post to the ```published_posts``` array.

### Minitest
Let's test our method with Minitest:

{% highlight ruby %}
require 'minitest/autorun'

class BlogTest < Minitest::Test
  def test_publish_adds_post_to_published_posts
    blog = Blog.new(author)
    post = Post.new
    blog.publish!(post)
    assert_includes(blog.published_posts, post)
  end
end

{% endhighlight %}

If we run the test, it will fail. Why? Well, what is ```author```? As you can see
if we exclude author when creating the ```Blog``` object it will complain with an
exception, ```ArgumentError```. But, in the code that we are testing, ```author``` 
has no role to play. It basically does nothing. It is important to the class, but 
it's not important to the method under test.

This is a great place to use a dummy. Let's add the author using a dummy. Minitest 
and RSpec do not have the concept of a pure dummy, although RSpec has a trick
that you can use. 

If you need a dummy in Minitest, you can just use an ```Object```. Or ```nil```. 
Or an empty ```Hash```. Or.. whatever.

{% highlight ruby %}
require 'minitest/autorun'

class BlogTest < Minitest::Test
  def test_publish_adds_post_to_published_posts
    author = Object.new
    blog = Blog.new(author)
    post = Post.new
    blog.publish!(post)
    assert_includes(blog.published_posts, post)
  end
end

{% endhighlight %}

### RSpec

The same test, but using RSpec: 

{% highlight ruby %}
require 'rspec/autorun'

describe Post do
  describe "#publish!" do
    it 'adds the post to the published_posts array' do
      author = Object.new
      blog = Blog.new(author)
      post = Post.new
      blog.publish!(post)
      expect(blog.published_posts).to include(post)
    end
  end
end
{% endhighlight %}

Also, another trick in RSpec is to use a double. A double that doesn't to anything
is a dummy. Remember that this only applies to RSpec:

{% highlight ruby %}
require 'rspec/autorun'

describe Post do
  describe "#publish!" do
    it 'adds the post to the published_posts array' do
      author = double
      blog = Blog.new(author)
      post = Post.new
      blog.publish!(post)
      expect(blog.published_posts).to include(post)
    end
  end
end
{% endhighlight %}

Do not get confused by the ```double``` keyword. For now, just pretend that it's 
an object that does nothing. Just like ```Object.new``` or ```nil```.

## Stub

Stubs are dummies, with one notable difference. Stubs have methods that they 
respond to.  But, their methods do not do anything except they return a predefined 
value. This means that their methods do not have an effect, they just return a 
value which will drive the test through the production code.

Let's see some examples. We have the ```Blog``` class which has the ```publish!```
method. 

{% highlight ruby %}
class Blog
  attr_reader :published_posts

  def initialize user
    @user = user
  end

  def publish! post
    published_posts << post if @user.author?
  end

  # Moar methods...
end
{% endhighlight %}

How can we test the ```publish!``` method? Due to the check that occurs in the 
```publish!``` method, we need a stub ```User``` object, which will return true 
when the ```author?``` method is called on it.

### Minitest

Creating a stub in Minitest is dead easy. Let's write the test for the publish 
method:

{% highlight ruby %}
require 'minitest/autorun'

class BlogTest < Minitest::Test
  def test_publish
    post = Post.new
    user = User.new
    blog = Blog.new(user)
    user.stub :author?, true do
      blog.publish!(post)
      assert_includes blog.published_posts, post
    end
  end
end
{% endhighlight %}

Stubbing in Minitest is done by calling ```.stub``` on the object/class that you 
want to stub a method on, with three arguments: the method name, the return value 
of the stub and a block. The block is basically the scope of the stub, or in other 
words, the stub will work only in the provided block.

Another way to achieve the same, which is a bit weird, is to avoid creating 
a new ```User``` object, create a dummy and declare a stub method on it.

{% highlight ruby %}
require 'minitest/autorun'

class BlogTest < Minitest::Test
  def test_publish
    post = Post.new
    user = Object.new

    # Tha stub
    def user.author?
      true
    end

    blog = Blog.new(user)
    blog.publish!(post)
    assert_includes blog.published_posts, post
  end
end
{% endhighlight %}

Using this way, we define the ```author?``` method on the singleton class of the 
user object, which will return ```true```.

### RSpec

The same test, or spec if you will, in RSpec is quite easy to write as well.

{% highlight ruby %}
require 'rspec/autorun'

describe Blog do
  describe '#publish' do
    it 'adds the published post to the published_posts collection' do
      post = Post.new
      user = double(:author? => true)
      blog = Blog.new(user)
      blog.publish!(post)
      expect(blog.published_posts).to include(post)
    end
  end
end
{% endhighlight %}

As you can see, the RSpec syntax is easy to understand as well. We create a double,
which is basically a dummy (makes sense, right?) and we declare a stub on it. The stub
is the ```author?``` which will return a predefined value, or in our case ```true```.

## Spy

I hope you guessed it - a spy is a stub. It's just an object whose methods do nothing
and return a predefined values to drive the test. However the spy remembers certain
things about the way and occurance of it's methods being called. For example, sometimes 
you want to assert that a certain method has been called. Another time, you might 
want to test how many times a method has been called. 

That's where spies are used.

Again, let's see some code. Take these classes and methods for example:

{% highlight ruby %}
class Blog
  attr_reader :published_posts

  def initialize user
    @user = user
  end

  def publish! post
    if @user.author?
      published_posts << post 
      NotificationService.notify_subscribers(post)
    end
  end
end
{% endhighlight %}

Now, the ```NotificationService.notify_subscribers``` method will send a notification
email to all the subscribers of the blog when a new post is published. 

When testing this method, we are interested in two things:

1. The post will be added to the ```published_posts``` array, and
2. The subscribers will be notified of the new post.

Since we already tested the first case in the example before, we'll just focus on 
no. 2. Let's test this method. 

### Minitest

Minitest does not support spies by default, but as always, we can use another gem
that plays nice with Minitest.

[Spy](https://github.com/ryanong/spy) is a really simple gem that does exactly that - spies.

Let's add some spies using [Spy](https://github.com/ryanong/spy).

{% highlight ruby %}
class BlogTest < Minitest::Test
  def test_notification_is_sent_when_publishing
    notification_service_spy = Spy.on(NotificationService, :notify_subscribers)
    post = Post.new
    user = User.new
    blog = Blog.new(user)

    blog.publish!(post)

    assert notificaion_service_spy.has_been_called?
  end
end
{% endhighlight %}

Quite self-explanatory, isn't it?

### RSpec

The RSpec syntactic sugar is really nice and human. This is how we can create spies 
in RSpec and use them appropriately.

{% highlight ruby %}
describe Blog do
  describe '#publish!' do
    it "sends notification when new post is published" do
      notification_spy = class_spy("Invitation")
      post = Post.new
      user = User.new
      blog = Blog.new(user)

      # If you want to set the expectation before the action is done:
      expect(notification_spy).to receive(:notify_subscribers).with(post)

      blog.publish!(post)

      # If you want to assert after the action is done:
      expect(NotificationService).to have_received(:notify_subscribers).with(post)
    end
  end
end
{% endhighlight %}

Also, keep in mind that since spy has the functionality of a stub, it can return 
fixed/predefined values. If you need your spy to return a value, you can do this 
in RSpec very easily, by chaining the ```and_return``` method to the spy:

{% highlight ruby %}
describe Blog do
  describe '#publish!' do
    it "sends notification when new post is published" do
      notification_spy = class_spy("Invitation")
      post = Post.new
      user = User.new
      blog = Blog.new(user)

      expect(notification_spy).to receive(:notify_subscribers).with(post).and_return(true)

      blog.publish!(post)
    end
  end
end
{% endhighlight %}

## (True) Mock

Before we go over the true mock, we have to make sure we understand the difference 
between "mocking", "the RSpec mock" and a "True Mock".

The whole issue behind the word **mock** is that in the beginning people started 
using it for simillar (but not the same) things and the word got left hanging in 
the air without a proper defnition.

So, "mocking" is usually used when thinking about creating objects that simulate 
the behavior of real objects or units. 

On the other hand, RSpec also has the concept of a mock. From RSpec's documentation:

<em>
rspec-mocks helps to control the context in a code example by letting you set 
known return values, fake implementations of methods, and even set expectations 
that specific messages are received by an object.
</em>

As you can see, there's some nomenclature overlapping and/or disagreement. 

Now, throw away any knowledge you have about the word **mock**. The True Mock is 
a very interesting type of test object. Of course, it has all the functionalities 
that a dummy, a stub and a spy have. And, a bit more.

The True Mock is an object that knows the flow of the method under test, and the test
can ask the mock if everything went well, usually called "verifying". This means that
the test will rely on the True Mock to track what are the effects of the method under
test and what methods have been called.

Let's see an example. The code under test:

{% highlight ruby %}
class Blog
  attr_reader :published_posts

  def initialize user
    @user = user
  end

  def publish! post
    published_posts << post if @user.can?(:publish)
  end

  def delete post
    post.delete! if @user.can?(:delete)
  end

end
{% endhighlight %}

### Minitest

Minitest has this type of testing object - it's called (believe it or not) ```Mock```.

{% highlight ruby %}
class BlogTest < Minitest::Test
  def test_post_deletion
    user = Minitest::Mock.new
    post = Minitest::Mock.new
    blog = Blog.new(user)

    user.expect :can?, true
    post.expect :delete!, true

    blog.delete(post)

    user.verify
    post.verify
  end
end
{% endhighlight %}

We create new ```Minitest::Mock``` objects and we set an expectation that it should 
receive the ```can?``` and ```delete!``` method, respectively. These will return 
```true```. Although this seems like a stub (setting a return value), the last 
line is what it differentiates it from a stub. The test asks the mocks to verify 
that the expected method was called, which will result in the test passing/failing.

### RSpec

With RSpec, it's a bit different. RSpec does not have a ```Mock``` object, but it's 
jack-of-all-trades ```double``` can do the same trick. 

{% highlight ruby %}
describe Blog do
  it 'can remove a post' do
    user = double
    post = double
    blog = Blog.new(user)

    expect(user).to receive(:can?).with(:publish)
    expect(user).to receive(:can?).with(:delete)
    expect(post).to receive(:delete!)

    blog.publish(post)
    blog.delete(post)
  end
end
{% endhighlight %}

This is a bit ambigous, since it's half-way between spies and a true mock objects.
But if you look in all the examples above, RSpec's ```double``` can morph into
anything. It all depends on the programmer what (s)he want's to use it as. I guess 
that's why it's called ```double```, because it can be any type of a test double 
you want.

## Fake

Last but not least, the fake. The fake is basically a half-baked, half-working 
implementation of the test object. It is not production ready, but, it's enough 
functional for the test to use it.

The popular testing frameworks do not have the concept of a fake. This is due to 
that writing the (needed) implementation for the fake is left for the programmer 
to do.

You should avoid fakes because fakes are standalone objects which are not used 
in the production code, but, they will grow linearly as the system grows. This means
that in the long run they can turn into code that will be very hard to maintain.
Also, pretty much every test can be written with one of the aforementioned testing
objects. So, if you are ever using fakes, make sure you know what you are getting
into.

## Conclusion

Now, that we got to the end, I hope that this article helped you understand the
theory behind test doubles and their implementations in Ruby. Be careful
where and how you use these test objects. If you go overboard you can create 
isolation between your tests and your production code, which can render your 
test suite useless.

Wow, you got to the very end. Thanks so much for reading! 

