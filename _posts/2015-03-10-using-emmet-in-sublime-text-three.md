---
layout: post
title: "Using Emmet (in Sublime Text 3)"
category: articles
tags: [emmet, sublime, plugin]
---

Front-end development is fun. Well, backend is fun too (especially with Ruby and Rails), but I feel like
front-end development is more rewarding. I mean, you type couple of lines of code and you can see the result of
your code immediately in your browser. It's nice.

Although it's cool, what frustrates me the most about front-end development is writing HTML.
I guess it's just laziness, but I'd prefer if there was a more "programmatic" way of writing HTML.
All the angled-brackets and closing the tags frustrates me. One can argue that I should use HAML (and SASS). Well, I agree. But sometimes we don't have the luxury of using preprocessors.

So, I found out about [Emmet](http://emmet.io).
But, first things first, what is Emmet?
Emmet is a plugin for \<enter-your-favourite-text-editor-here\>.
It basically allows the developer to write HTML and CSS very fast!
Of course, once you get used to using it.

Problem is, I found out about it like two years ago, but I never gave it a try.
So last night, I finally gave it a try and oh boy, what a great surprise I got! Which lead to this:

<blockquote class="twitter-tweet" lang="en"><p>I feel ashamed that today I finally tried it and found out that <a href="https://twitter.com/emmetio">@emmetio</a> is an awesome tool (knowing of it&#39;s existence for couple of years).</p>&mdash; Ile (@fteem) <a href="https://twitter.com/fteem/status/564948599512264704">February 10, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Yup, I am ashamed I have never tried it out before, although I knew about it's exsistence
for (literally) years!

So, go ahead and [install it](http://emmet.io/download/).
After you are done I will show you couple of cool ways you can use it to boost your productivity.

You done? Awesome!

## The simple stuff

How many times have you written a boilerplate HTML5 template?
You know, the very boilerplate doctype, a html tag, head and body tags?

This is easily doable using Emmet. Just type:

{% highlight html %}
html:5
{% endhighlight %}

And hit TAB. One **important** thing, and this applies to all commands - make sure
the cursor is after the last character of the command, on the end of the line.

![html:5 command](http://i.imgur.com/nhlDnqd.gif)

Awesome, right? After you type html:5 just hit the tab button and BOOM, you have a boilerplate HTML snippet
ready for you to start working on it.

## Lorem ipsum
What is HTML without some dummy text? Emmet got us covered.

If you want a simple and short lipsum text, type **lorem** and press tab.
Also, if you want a dummy text with a specific length, you can type **lorem100**
(for dummy text of a hundred words) and press tab.

## Child and sibling elements

So, as we know, nesting elements is in the nature of HTML. What does Emmet do when
it comes to HTML elements nesting?

Lets create the following HTML using Emmet.

{% highlight html %}
<div>
  <ul>
    <li></li>
  </ul>
</div>
{% endhighlight %}

It's really simple.

Just type:

{% highlight html %}
div>ul>li
{% endhighlight %}

and press tab.

![children](http://i.imgur.com/V2jLGhS.gif)

Easy, right? Lets try writing an unordered list with three list items in it:

![list1](http://i.imgur.com/1rBW9V6.gif)

So, the **>** operator tells Emmet that the following element will be a child element of the element before the operator.
The **+** operator tells Emmet that the element that follows will be a **sibling** of the element before the operator.

What happens when we go way deep into the children elements and we need to go up to the parent element,
to create a sibling of the parent?

Again, easy!

Let's create two lists, one next to another, wrapped in a div:

![list2](http://i.imgur.com/806uEpX.gif)

This can be easily achieved using **groups**. Just like in maths, or programming languages,
grouping of expressions are done using parentheses.

![list3](http://i.imgur.com/yaayuzs.gif)

As you can see, now we have two groups which are literally the same. Wouldn't it be
really cool if we can repeat the first group twice?

## Classes and IDs

But hold on, lets see how we can add classes and ids to HTML elements using Emmet's syntax.

You can add a class to an element by adding .class-name to the element and #id to add an id to it.
Easy as that.

Also, if you add an id or a class to a div, you can omit the div and just do .class-name or .id.
Emmet will know that you want to create a div with that class/id.

For example:

![classes and ids](http://i.imgur.com/DqofzYn.gif)

## Multiplication

So, back to our earlier example - two lists wrapped in a div. But this time let's write it using multiplication:

![list4](http://i.imgur.com/HOvtU4w.gif)

Really cool, right? The **\*** operator is just like multiplication in maths - it repeats the
expression (in our example, the group) two times.

I know what you are thinking now - what's the use of writing li+li+li three times when we can
use repetition to do this? Check this out:

![list5](http://i.imgur.com/e5kbczo.gif)

BOOM! Much shorter.

We can combine multiplication and classes/ids in a very cool way.

When applying multiplication to a element or a group, the **$** operator allows
us to apply the number of each multiplication in the class or id name.

For example:

![dollar-sign](http://i.imgur.com/WGSQIFG.gif)

## Adding text and attributes to HTML elements

I think that an input field would be a great example for this.

{% highlight html %}
  <input type="text" placeholder="Enter your name" />
{% endhighlight %}

The syntax for adding attributes to elements in Emmet is very simillar to the one CSS selectors use:

![attributes](http://i.imgur.com/tUsmBkm.gif)

And adding content is done with appending **{ some content }** to the element.

![content](http://i.imgur.com/EhedIG9.gif)

## Moar?

Emmet has more cool features. For example, it has a [fuzzy search](http://docs.emmet.io/css-abbreviations/fuzzy-search/)
for CSS and [other actions](http://docs.emmet.io/actions/).
You can read more about it in [Emmet's docs](http://docs.emmet.io/). And make sure
you check out the [cheatsheet](http://docs.emmet.io/cheat-sheet/) - it's very useful!
