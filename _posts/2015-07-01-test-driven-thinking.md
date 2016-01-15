---
layout: post
title: "Test-Driven Thinking"
tags: [testing, tdd]
---


There has always been a lot of noise about Test-Driven Development (TDD), best practices 
and it's pros and cons. I don't know if you remember, but last year DHH, Kent Beck 
and Martin Fowler had a series of public hangouts where they shared their opinions on TDD and
testing in general and engaged in a very interesting discussion. These hangouts were recorded 
and in my opnion - they are a [must watch](https://www.youtube.com/watch?v=z9quxZsLcfo).

But, this blog post won't be about TDD itself. Disclaimer: I really like TDD. Very often 
it's joy to practice TDD. Sometimes it's very challenging, but in my opinion, challenge means growth.

## Experience
We, software developers, very often rely on our experience. Which is good. I am not saying using your experience and prior knowledge is bad. No. On the contrary, it's good. What I find amusing is that we think that similar problems can be solved with the same solution. For example, in the past we have worked on a problem where adding a caching layer to the application sped up the application which solved the problem. What we often do is we would recognize a similar problem in our current work and we think that using a caching layer would do the trick. And very often, we are wrong. And we smash our head in a wall.

I've found myself into the same position when practicing TDD. In the beginning - even way too often.

## The TDD jihad [1]
Quote often, when I practice TDD, I find myself in a position where I don't know what I should do next. What I am saying is - I am basically stuck. And I have indentified various reasons for this. But almost always, the main reason is: lack of domain knowledge. It's like, you stumble into a part of the app you are building and you don't have a slightest idea of what direction you should go next. And very often I think about how I can circumvent this.

So what I do in this situation? Well, usually I get overwhelmed and I need a break. I get up and make myself a cup of coffee. Then I sit down, take a sip of the coffee and read what I've done until then. I am pretty sure we've all been in this situation, more or less.

What I always realize is that I am over-focusing on making those tests pass. One would argue that this is the whole point of TDD. But I'd say no!

## Test-driven thinking
While we are writing those failing tests and then adding the implementation code (that will make the tests green) we are missing a very interesting thing - those tests are talking to us. Our test suites contain lots of knowledge about the domain model. We are writing various tests cases which can later be used as specification detail for the system itself (hence the name 'specs'). By adding all of that knowledge about the domain into our test suite we are locking that knowledge and tying it do the application, so the tests are basically a bio of the domain.

I have colleagues that when they start working on software, they always start with the tests. It's not about the TDD cycle. Actually, it's about all of that knowledge that can be found in the code's test. The tests (if well written) can tell a lot about the class and it's implementation. And they immediately get the point of that piece of code and it's purpose. Now, that's what I'd call test-driven thinking. You know your destination (the final needed functionality of the code), you let the tests talk to you, you think about to where you've come to and where you need to be heading.

Instead of lurking around with no idea where to go next - you must have the end goal in mind & you must read what you've already implemented. And the tests will show you the end of the tunnel!

Material Exploration
And what about when you get parachuted in a legacy project that deperately needs an upgrade and you literally haveno clue what to do? I've been into this position and believe me, it sux!

Well, take small steps! If you take small steps, you iterate and eventually you'll get to the perfect functionality of your code. Easier said than done? Of course. It's software development! :-)

Not having a clue is OK. No, really, it is. No one knows everything. But the key here is to take smalls steps. This means that while you work you can mess with the code. And by code I mean the tests. Don't touch the implementation until you are confident. If there's a part of the app that does not have tests - write them! By doing that you'll explore the domain, the code, the implementation and, most importantly, you learn. The domain model will sinks into your head, you'll start to understand it and eventually you'll be in a good position to start refactoring. That is called material exploration. You learn from your work while you work.

## Outro
So, yeah, whenever you get stuck - don't worry. Just take small steps and don't rush. Read the existing tests, add more, then add some more and build up your confidence about the code. After a while, you'll be very confident about making changes and adding new functionality.

[1] Jihad means struggle. Not holy war.
