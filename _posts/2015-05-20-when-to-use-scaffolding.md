---
layout: post
title: "When to use scaffolding in Rails?"
tags: [ruby, rails, scaffolding]
featured_image: /images/cover1.jpg
---

Having about 2.5 years of experience with Ruby on Rails, I finally got the chance
to be a mentor. The person I mentor is a friend that I know from university and we
have a healthy relationship. When you are a mentor, you have to have that type of
relationship. Although at the beginning I had a lot of enthusiasm, I have to say mentoring
is very challenging and often very frustrating. So, as my colleague was learning the basics of Rails
one of the questions I got from him is "When should I use scaffolding?".

It seems to me that beginners really dig scaffolding. Of course, who wouldn't like
their code to be generated instead of writing it by hand. But at first, I thought
the answer to this is obvious. Unfortunately, clearly it's not. So I sat down and
made a list of when and where scaffolding is really useful.

## For beginners

A lot of the introductory books about Rails start with scaffolding. They usually
build a simple blog system in a chapter. I agree, this has a purpose and I think that
it's a nice introduction to Rails generators. But, in my honest opinion, it's better to
get down to writing your own code and understand how everything works instead of
relying on generated code.

## Simple CRUD applications

As a somewhat experienced Rails developer, I can admit that scaffolding is perfect
for tiny CRUD applications. The problem is that in the real world no one will pay you
for a tiny CRUD app. The end goal is always far far away from simple CRUD. Keep in
mind that scaffolding supports you as you build the actual product - it clearly can not
create the actual product.

## Prototyping

Scaffolding allows rapid prototyping. I don't know how many of you remember the
"20 minutes blog" video by DHH, but that's what exactly is happening. When using
scaffold for prototyping, you have to be concious that it's a prototype that you are building.
And prototypes have a purpose, and once they fulfill it - you throw them away. Well,
you might not need to throw the whole thing away, but you will surely delete the
unneeded stuff.

## MVP or POC

Another point is that scaffolding is great for building an MVP (Minimum Viable Product)
or a POC (Proof Of Concept). Yes, this point has overlapping with the other points here,
but in rare cases you can create an MVP or a POC by using scaffolding. The problem is,
in most cases an MVP is far from a simple CRUD app that scaffolding can take care of.

## Takeaway

As a beginners, you can learn a lot from scaffolding. For example, the scaffolded controllers
are nicely done and you can see how usually CRUD controllers look like. But, to any
beginner I would recommend to get to know the Rails generators in general. In that way,
much of the files needed for the app can be generated for him, but the code (which is what matters the most)
will be written by the programmer. At the end, keep in mind that regardless if you write the code,
or a colleague writes the code, or a machine generates the code - you will maintain that code
very possibly for a long time. Because software is built to grow, not die.

