---
layout: post
title: "Refactoring in Ruby: TDD your way through Extract Class"
tags: [ruby, refactor, tdd]
---

There are a lot of refactoring patterns available out there for us. I assume that
most of us use these patterns, at certain times without being aware that those
refactoring steps are defined as a pattern in the past. In this post, I will take
you through an example of refactoring Ruby code with the Extract Class pattern
by using Test-Driven Development.

Let's dive in!

## What went wrong at the test?

Say we are professors in university and we compile a CSV file with our students'
grades. Some of them are good, some are not that good. We want to have a small
informal meeting with the students that did not pass the exam, in a classroom
and review the tests and discuss what went wrong. Given that there are a lot of
students in our class, writing an email for each of the students is painful.
Well, good we are programming professors, so we can write our own little program
to automate sending those emails:

{% highlight ruby %}
require 'csv'
require 'mail'

class Parser
  def initialize(file_path)
    @file_path = file_path
  end

  def parse!
    CSV.foreach(@file_path, headers: true) do |row|
      if failed_test?(row["points"])
        Mail.deliver do
          to      row["email"]
          from    'John Doe <john@doe.com>'
          subject 'Post exam meeting invitation'
          body    failed_exam_body(row["first_name"], row["points"])
        end
      else
        Mail.deliver do
          to      row["email"]
          from    'John Doe <john@doe.com>'
          subject 'Exam results'
          body    passed_exam_body(row["first_name"], row["points"])
        end
      end
    end
  end

  private

  def failed_test?(points)
    points < 50
  end

  def failed_exam_body(student_name, points)
    %Q{
      Hi #{student_name},

      On the exam last week you got #{points} points.
      It seems to me that you ran into some problems with it so I would like to
      invite you to a meeting in classrom 201, tomorrow at 2PM.

      Regards,
      Prof. John Doe
    }
  end

  def passed_exam_body(student_name, points)
    %Q{
      Hi #{student_name},

      On the exam last week you got #{scored_points} points.

      Congratulations,
      Prof. John Doe
    }
  end
end
{% endhighlight %}

As you can see, it's quite a simple class. It takes a CSV file path and parses
it. For each row it checks if the points are less then the minimum amount to
pass the exam. If so, then a customised email is sent to each of the students.

Now, if we take a moment to think about the design of this class, there's a
question that we can ask ourselves: what does this class do? Although the
question is simple, the answer can reveal something very interesting.

The class parses a CSV file **and** sends an email to each of the students that
failed the test. You see, the word "and" is bolded because it points out to a
code smell - violation of the Single Responsibility Principle. If you don't know
what SRP is, I recommend you check out my
[blog post](/solid-principles){:target="_blank"} on SOLID principles.

So, how can we refactor our class? Well, one of the options is using the Extract
Class pattern. Let's check it out.

## Extract Class pattern

This pattern falls into a group of patterns that are used for moving features
(or functionalities) between objects. The benefit of this refactor is to make
your classes adhere to the SRP. Classes that adhere to the SRP are much easier
to understand and expand onto.

Now,back to the design of the class. Since we agreed that it breaks the SRP, we
need to decide which functionality we will move out into a new class. For our
example, we will let the `Parser` class just parse the CSV, while we will create
a new class called `Exam` which will contain all of the data and behaviour
around the exam data that we parse from the CSV. After that, we will create a
`Mailer` class, whose role will be just to send the email.

With that, we will have three classes into play - all having just **one
responsibility**, whether is representing an `Exam`, parsing a CSV or sending
emails.

The first step will be to wish the interface of the `Exam` and `Mailer` classes
and use them in the `Parser` class. This is called "programming by
[wishful thinking](http://c2.com/cgi/wiki?WishfulThinking)".

{% highlight ruby %}
# parser.rb
require 'csv'

class Parser
  def initialize(file_path)
    @file_path = File.expand_path(file_path)
  end

  def parse!
    CSV.foreach(@file_path, headers: true) do |row|
      exam = Exam.new(row)
      if exam.failed?
        ExamMailer.failed!(exam)
      else
        ExamMailer.passed!(exam)
      end
    end
  end
end

Parser.new("grades.csv").parse!
{% endhighlight %}

Good. As you can see, we've added the new classes in `Parser#parse!` method
which radically simplified the readiness and simplicity of it. Since we now
have the classes in place, let's take a step back and use TDD to create these
classes.

## Introducing the Exam

As we said before, the `Exam` class will hold the data and represent the
behaviour of an exam. But what does that mean? Well, simply put, it will have
methods that will decide if an exam is failed based on the data (points). Let's
TDD our way through this and write some tests before we implement the class:

{% highlight ruby %}
# exam_test.rb

class ExamTest < Minitest::Test
  def test_it_can_be_failed
    exam = Exam.new({ 'points' => '20' })
    assert exam.failed?
  end

  def test_it_can_be_passed
    exam = Exam.new({ 'points' => '51' })
    assert exam.passed?
  end
end
{% endhighlight %}

I assume the tests are quite self explanatory. The constructor will take a hash
as an argument and extract the points from it. Then the `Exam#passed?` and
`Exam#failed?` methods will check if the exam is passed or failed. The rule here
is that an exam is failed if it has less than 50 points scored. Let's implement
this:

{% highlight ruby %}
# exam.rb

class Exam
  attr_reader :scored_points

  def initialize(entry)
    @scored_points = entry['points'].to_i
  end

  def failed?
    @scored_points < 50
  end

  def passed?
    !failed?
  end
end
{% endhighlight %}

Really easy, right? If we run the tests, they will pass. Additionally we will
need the exam to contain the data for the student which has taken the exam. Some
of you might think that a student should be a separate object and I would say
that you are right. Also, others might say that there can be separate classes
for a failed and passed exams.

But, for this example we can keep it really simple. Let's add some tests for the
attribute readers:

{% highlight ruby %}
class ExamTest < Minitest::Test
  def test_student_name
    student_name = 'Ilija'
    exam = Exam.new({ 'first_name' => student_name })

    assert_equal student_name, exam.student_name
  end

  def test_student_email
    student_email = 'ile@eftimov.net'
    exam = Exam.new({ 'email' => student_email })

    assert_equal student_email, exam.student_email
  end

  def test_failure
    exam = Exam.new({ 'points' => '20' })
    assert exam.failed?
  end

  def test_success
    exam = Exam.new({ 'points' => '51' })
    assert exam.passed?
  end
end
{% endhighlight %}

Implementing these methods is quite easy as well:

{% highlight ruby %}
class Exam
  attr_reader :student_name, :student_email, :scored_points

  def initialize(entry)
    @student_name  = entry['first_name']
    @student_email = entry['email']
    @scored_points = entry['points'].to_i
  end

  def failed?
    @scored_points < 50
  end

  def passed?
    !failed?
  end
end
{% endhighlight %}

That is our `Exam` class with all of it's glory. Did you think about what we did
in this step? Really simple stuff - noticing that there is room for a class to
encapsulate all of the required data so it can answer to the question "is this
exam failed or not?". We first wished what the interfaces of our classes should
be and then we created the first class, using TDD.

This is the **Extract Class** pattern at it's core - reducing complexity and
improving readibility by extracting data and behaviour to a new class.

That would be it at this moment. The `Exam` holds enough behaviour and data so
we can continue with the other classes.

## Building the mailer

The next step is the `ExamMailer`. The mailer should contain only the behaviour
required to send the emails - nothing else. As we decided in the first step, the
mailer will have two methods: `ExamMailer.passed!` and `ExamMailer.failed!`. The
mailer class will use the
[Mail gem](https://github.com/mikel/mail){:target = "_blank"} to send emails.
Let's write some tests around them first and implement them after:

{% highlight ruby %}
Mail.defaults do
  delivery_method :test
end

class MailerTest < Minitest::Test

  def setup
    Mail::TestMailer.deliveries.clear
    @student_mail = 'john@doe.com'
    @student_name = 'John'
  end

  def test_failed_exam_mail
    points = '49'
    exam = Exam.new({ 'points' => points,
                      'email' => @student_mail,
                      'first_name' => @student_name })
    ExamMailer.failed!(exam)
    mail = Mail::TestMailer.deliveries.first

    assert_equal @student_mail, mail.to.first
    assert_equal "Post exam meeting invitation", mail.subject
    assert_match /you got #{points} points/i, mail.body.raw_source
  end

  def test_passed_exam_mail
    points = '50'
    exam = Exam.new({ 'points' => points,
                      'email' => @student_mail,
                      'first_name' => @student_name })
    ExamMailer.passed!(exam)
    mail = Mail::TestMailer.deliveries.first

    assert_equal @student_mail, mail.to.first
    assert_equal "Exam results", mail.subject
    assert_match /you got #{points} points/i, mail.body.raw_source
  end
end
{% endhighlight %}

Now, there's a bit of boilerplate code here, which deals with the Mail gem. In
the beginning of the file, we set the delivery method of the Mail gem to be
`:test`. This means that the gem will hold all of the emails that are sent in
memory, without trying to make an actual delivery. Also, in our `setup` method
we invoke the `Mail::TestMailer.deliveries.clear` method, which clears the
deliveries collection before running each test. This is done so we have a fresh
collection of sent emails for each test, which helps with avoiding any
confusion.

As you can notice in the tests, we assert on the recipient email, on the e-mail
subject and on the e-mail body. With this step, we want to make sure that the
most essential parts of the e-mail (recipient, body and subject) are correct.

Extracting the mailer logic from the initial script to a new mailer class is
rather simple:

{% highlight ruby %}
require 'mail'

class ExamMailer
  def self.failed!(exam)
    new(exam).failed!
  end

  def self.passed!(exam)
    new(exam).passed!
  end

  def initialize(exam)
    @exam = exam
  end

  def failed!
    send_mail!('Post exam meeting invitation', failed_exam_email_body)
  end

  def passed!
    send_mail!('Exam results', passed_exam_email_body)
  end

  private

  def send_mail!(subject, body)
    mail = Mail.new
    mail.to = @exam.student_email
    mail.from = 'John Doe <john@doe.com>'
    mail.subject = subject
    mail.body = body
    mail.deliver
  end

  def passed_exam_email_body
    %Q{
      Hi #{@exam.student_name},

      On the exam last week you got #{@exam.scored_points} points.

      Congratulations,
      Prof. John Doe
    }
  end

  def failed_exam_email_body
    %Q{
      Hi #{@exam.student_name},

      On the exam last week you got #{@exam.scored_points} points.
      It seems to me that you ran into some problems with it so I would like to
      invite you to a group meeting in classrom 201, tomorrow at 2PM.

      Regards,
      Prof. John Doe
    }
  end
end
{% endhighlight %}

Most of the magic lies within the `send_mail!` method. It builds a new `Mail`
object and delivers it.

As you saw in this step, we extracted the mailing logic from the script to a new
well-defined class. The next (and last step) is reviewing the original `Parser`
class and pondering a bit with it.

## Back to the parser

After we did all of the hard work around extracting two new classes and writing
tests for them, it is time to take a look at the `Parser` class. I would say
that improving it further would be over-engineering it, at least in the scope of
this article. The only thing that I would like us to think about is the name of
the class. Does `Parser` do it?

This is quite subjective and I would guess some of you wouldn't agree with me,
but in my mind the `Parser` is quite broad term. In this instance, I think that
`Grader` is a better name for it.

## Taking a step back

As you could see in this article, there's quite a bit of work around refactoring
existing code. There are more than a two dozen patterns around refactoring, with
Extract Class being one of the simplest ones. Having this said, I would like to
ask you a question:

Do you really need to think in patterns? Do we really need to have a
"predefined sequence of steps" to make our code better?

I think that it's not really about applying a certain pattern, it's more about
caring about the design of the code and writing tests for it. As a rule of thumb,
I always try to think about the five
[SOLID principles](/solid-principles/){:target="_blank"} when writing my code.
And when you are refactoring - understand the motivation and the context behind
the code that you are refactoring, without focusing on which pattern you will
apply.

