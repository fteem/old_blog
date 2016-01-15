---
layout: post
title: "Template Method Pattern in Ruby"
tags: [angularjs, google, javascript]
---
When working as a software developer, knowledge of some design patterns is always welcomed. If you've never heard about design patterns, they are basically some general reusable patterns for common problems that developers run into. There's a big list of these and knowing all of them is a bit hard. Well, hard might not be the right word, but it takes **a lot** of practice to master them all. Lets take a look at one of the (in my opinion) easier patterns - the Template Method Pattern and implement it in Ruby.

### Template method pattern

The Template method pattern (or TMP) is a design pattern that defines the program skeleton, or an algorithm skeleton in a method, and the methods that this algorithm is dependent on. The key thing is that some/all of the methods are deferred to subclasses. If this is a bit confusing, bear with me, I promise the example will clear it up. :-)

### A simple payment system

Lets say we are working on an e-commerce web application. Like every e-commerce webapp we need a payment system in place for a great user experience. Usually, the payment process has the same flow regardless of the payment provider used.

Keep in mind that this is an **oversimplified example** and payment systems in real web applications have more complexity around them. The flow of our example payment system is:

1.  Authenticate the merchant (the application) with the provider.
2.  Send the user's card data to the provider with the amount of the order.
3.  Receive confirmation/error from the provider.

If we want to implement this payment algorithm using Stripe, it would look something like:

{% gist 5cc8b7344910ea4bdf90 %}

Again, keep in mind, this is just dummy code, not actual Stripe payment gateway implementation. Okay, so this looks all fine. We can instantiate a new _StripePayment_ object, pass the card object and the amount of the order as parameters in the initializer and call the _process_payment!_ method on the object to execute the payment. For a successful payment, we need the Merchant (our web application) to successfully authenticate with the payment provider (Stripe) and then the credit card to be charged the total of the order. If any of these two fail, the payment wont be processed.

### What about PayPal?

Software is made to grow, not die. Or sometimes, get rewritten. So what if our customers need us to add PayPal as a payment option? Easy, right? We can just add another class called _PaypalPayment_
and add the payment logic in the class. But, hold on for a second! What if we need to add Skrill as a payment too? And what if we need to add simple credit card payment, because we have customers that don't want to register accounts with any of the aforementioned payment providers? You see, we run into a problem. We will have to add separate classes that will share a lot of the functionality. The Template Method Pattern can come into play here. So, how can we leverage TMP? We need to create a _BasePayment_ class that will store the **template method****(s)**. Then, we will create subclasses, one for every payment provider we use. In the subclasses we will add the specifics of each payment provider, while on the surface the payment will be done by calling the _process_payment!_ method on the payment object, regardless of its class. Let's create our _BasePayment_ and _StripePayment_ classes and see how we can leverage TMP.

{% gist 8a7073359b7cbcb2b86a %}

As you can see, the template-method-holding-class, or _BasePayment_, cannot be used as a standalone class. This is because we want to use the class just as a blueprint for the subclasses, instead of instantiating any objects of it. Also, all of the _BasePayment_ subclasses will have to implement the _authenticate_merchant_ and _make_payment_ methods before they can be usable. Let's add PayPal as a payment option now!

{% gist 2bbb3233a1e2ab0d10df %}

As you can see, although the subclasses look alike, the logic in the template methods is very provider specific. The advantage is that the shared logic is inherited and the interfaces of all the payment classes are the same while the algorithm of the authentication and the payment is different in the subclasses. This is how TMP can be done in our example. Now, adding Skrill, or some other payment provider is really easy. Also, another advantage is that the rest of the web application can easily work with any of the payment classes.

### Outro

I hope my examples explained the Template Method design pattern. You can use the TMP when there are multiple invariants of a type in your code, and you leave placeholders in your parent type so you can easily customize the subtypes. Also, this enables the developer to not break the functionality of all the subtypes that they inherit from their super-type. Basically, you should use this pattern when you want to have a varying algorithm in a class. The template method holder class should implement the skeleton, while the subclasses should implement the details of the varying algorithm. Questions for you - in Ruby, which approach to TMP do you use? Are there more ways to achieve TMP? Or maybe you have some feedback for this post? Please share your thoughts in the comments. Thanks for reading!  


**Update:** Updated the code to raise [NotImplementedError](http://ruby-doc.org/core-2.2.0/NotImplementedError.html). Thanks to [David](http://rubylogs.com/template-method-pattern-in-ruby/#comment-13) and [serg-kovalev](https://gist.github.com/fteem/8a7073359b7cbcb2b86a#comment-1432191) for the suggestion!
