---
layout: post
title: "Why and how to test Rake tasks in your Rails application"
tags: [rails, testing, rake, tasks]
---

Most of us write some Rake tasks and completely forget about them. In fact, we
rarely give any love to our Rake tasks. Well, I think it's time we change that.
Let's see why and how we can test our Rake tasks.


## But, why?

Yes, it's a legit question. You can always say "I already tested my classes!".
But, there are couple of reasons why you should always test your Rake tasks:

1. Rake tasks is code. And all code should be tested.
2. Rake tasks can use any part of your app. Models, service classes and what not.
    If one of the classes that the Rake task relies on changes, you have to know if
    it will break it.
3. Rake tasks can do heavy lifting. Perhaps you have a cron on Heroku that runs an
    email campaign that calls a Rake task. Or you generate reports with a Rake task. This
    is important and you need to know if it actually works.
4. Forgetting about your Rake tasks is easy. Or, if you inherit a codebase it's easy
    to not even notice them when beginning the project. Rake tasks are code as well
    and breaking it is easy.

So, yeah, test your Rake tasks!

## The Rake task

For our example application, let's imagine we have an application where users can
register. But, just like with most of the websites online, people sometimes
register but they do not finish the registration process. Often, the database
can be polluted with records of unfinished user registration.

The application has a Rake task that deletes users that haven't finished their
registration process.

{% highlight ruby %}
# lib/tasks/users/remove_unconfirmed.rb
namespace :users do
  desc "Delete users that have not finished the registration process"
  task :remove_unconfirmed do
    User.unconfirmed.each {|user| user.delete }
  end
end
{% endhighlight %}

This task is run with as a cron job. Let's see how we can test it.

Lately, I prefer testing with Minitest. I like it because it's really tiny, quite
verbose, magic-less and it's pure Ruby. Now, since Test::Unit is basically
syntactic-sugar on top of Minitest, I will show you how to test the Rake task
with it.

## Integration test

We will take for granted that the `User` model and the `UserMailer` are already
tested. Our next objective is to test this Rake task. We will need one test for
this task - it will check that the task deletes the unconfirmed user. This is
basically an integration test that will work with actual records in the database.

Whenever you are testing Rake tasks, you need to load the tasks from the Rails
application itself. Note that in your tests you should change `MyApplication` to
the name of your application.

The test will look like this:

{% highlight ruby %}
# test/lib/user_notify_test.rb
require 'test_helper'

class RemoveUnconfirmedTaskTest < ActiveSupport::TestCase
  setup do
    @confirmed_user = User.create(email: "john@appleseed.com", confirmed_at: Time.now)
    @unconfirmed_user = User.create(email: "jane@doe.com", confirmed_at: nil)
    MyApplication::Application.load_tasks
    Rake::Task['users:remove_unconfirmed'].invoke
  end

  test "unconfirmed user is deleted" do
    assert_nil User.find_by(email: "jane@doe.com")
  end

  test "confimed user is not deleted" do
    assert_equal @confimed_user, User.find_by(email: "john@appleseed.com")
  end
end

{% endhighlight %}

Now, in the `setup` step, we create two users - one which will be confirmed and
one unconfirmed. Then, we load the tasks from the application. This is done by
adding:

{% highlight ruby %}
MyApplication::Application.load_tasks
{% endhighlight %}

This task basically requires rake and loads each file in the `lib/tasks` path.
You can see it's source [here](https://github.com/rails/rails/blob/0450642c27af3af35b449208b21695fd55c30f90/railties/lib/rails/engine.rb#L455). Next, we invoke the task itself. We do this by adding:

{% highlight ruby %}
Rake::Task['users:remove_unconfirmed'].invoke
{% endhighlight %}

in our test. This line locates the task by it's name and returns a `Rake::Task`
object. Then, we call `invoke` on it, which executes the task. After that we
make assertions on the data in the database, expecting the unconfirmed user to
be removed from the database.

## Unit test

Another approach at testing this is exctracting the logic from the Rake task to
a utility class. Then, we would call the method from the class that contains the
logic in the Rake task.

Let's create a `UserRemoval` class, with a class method called
`remove_unconfirmed`.

{% highlight ruby %}
class UserRemoval
  def self.remove_unconfirmed!
    User.unconfirmed.each {|user| user.delete }
  end
end
{% endhighlight %}

Then, our Rake task would look like:

{% highlight ruby %}
# lib/tasks/users/remove_unconfirmed.rb
namespace :users do
  desc "Delete users that have not finished the registration process"
  task :remove_unconfirmed do
    UserRemoval.remove_unconfirmed!
  end
end
{% endhighlight %}

Since all of the logic in the rake task is contained in another class, we can
test the class itself, instead of testing the task. This is quite trivial, since
we can pretty much use the same test, with some small changes.

The test would look like this:

{% highlight ruby %}
# test/lib/user_notify_test.rb
require 'test_helper'

class UserRemovalTest < ActiveSupport::TestCase
  setup do
    @confirmed_user = User.create(email: "john@appleseed.com", confirmed_at: Time.now)
    @unconfirmed_user = User.create(email: "jane@doe.com", confirmed_at: nil)
    UserRemoval.remove_unconfirmed!
  end

  test "unconfirmed user is deleted" do
    assert_nil User.find_by(email: "jane@doe.com")
  end

  test "confimed user is not deleted" do
    assert_equal @confimed_user, User.find_by(email: "john@appleseed.com")
  end
end
{% endhighlight %}

As you can notice, the test is basically the same, without loading/invoking the
Rake task. Instead, we just call the `UserRemoval.remove_unconfirmed!` method.

Now, we could stub out the calls to the database and isolate the tests from
database IO, but I usually prefer to do these cheap tests with real databse
records.

## Outro

Now, I know that this is quite a beginner tutorial on testing Rake tasks and I
am sure it didn't rock the world of someone adept at Ruby and testing. But, I
have been asked on couple of occasions about this and have seen some questions on
Stack Overflow about it so I figured it would be nice to document it somewhere.

What do you think about testing Rake tasks? Do you test them? If you do, do you
write integration tests or maybe you prefer unit tests with mocks? Or, maybe you
take another approach?

Let me know in the comments!


