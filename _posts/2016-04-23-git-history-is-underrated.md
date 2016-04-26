---
layout: post
title: "Git history is underrated"
tags: [git, history]
image: git-history-underrated.png
---

Most of us use Git on a daily basis. We have all read a book (or part of a book)
about Git, we learned how to do commits, track additions and removals, work with
branches and so on. But, there's so much more to Git than just committing your 
changes. In this post I am going to rant a bit about how we don't utilise the
power of our Git history, and how one can actually start doing it.

## What is history, actually?

Well, you see, the word history comes from the Greek word "historia", which means
"inquiry, knowledge acquired by investigation". The modern definition of history
is "study of the past, particularly how it relates to humans". So, while we have
history books about what has happened to humans in the past, Git is the only
tool we have to see the history of our source code.

While in some parts, especially in the early years of humanity, history can be
misguiding or unclear, the history of a codebase is very clear from the very 
beginning. Or is it? Think about this - if we could be able to go back in time, 
and watch every single thing happen and write all of these things down, how 
different would the world be? We would know all of the facts, instead of 
assumptions and relying on some manuscripts from thousands years ago.

So, when we are given the chance to start a new Git repo, and do the legendary
initial commit, why don't we care about what the history of the codebase will
look like?

## Commit messages

Picture this - you are writing this awesome feature at work, which will have a
huge impact on the product. Of course, a high-impact feature almost always means
high impact on the codebase. Thousands of deletions and additions, a huge diff,
sits on a feature branch. The code has been well tested and it's about to be 
merged in `master`. You press the button, and couple of minutes later it's on
production. Everyone on the team is monitoring the application and it seems that
you've done a marvelous job. Time for beers, right? Of course!

Fast forward six months ahead, a bug pops out. You start debugging and see that
there might be some edge case bug that it's crazy hard to debug. After some
debugging, you have to see when the "smelly" files were changed. You do:

```bash
git log -p /path/to/smelly/file
```

While scrolling through the history you see your squashed merge commit from 6
months ago. You grab the commit hash, and do:

```bash
git show dc3730f72d46de2bffa62bc3e9203e4d0f4210c5
```

You say to yourself: "Oh, yes, I remember I did this. The deployment went so 
smooth without any bugs and we all felt like champions. Great, I should be able
to find my way through this easily!" 

But, can you?

## Context is king

So great, you found your merge commit. But, how are you going to figure out the
context of the diff? Why did some method's signature change? What was wrong with
the naming of that class that you had to change it to something else? Also,
you added a new table and couple of polymorphic associations? Whoa, this must've
been quite a bit of work. But only one sqashed merge commit.

Tough stuff right? There's no way to regain the context easily. Well, this is 
where Git history *should* shine! While there are a ton of good articles on
writing good commit messages, we usually go with "fixed a bug", "typo" or "a 
very long commit message that actually doesn't make sense". **Your repo's history
is there to help you regain context and understand a change that has been done
by someone at a point of time.** It's easy as that. And there's a plethora of
options that can be applied to the `git log` command which can be very helpful
in achieving that. 

So, to make it easier for yourself and other people to regain context easily,
be careful of what and how you commit.

## An ideal history

Let's be frank - there's no ideal history, but an optimal one is sure possible.
If you would ask me, it falls down to two things: clean and concise commit 
messages and small diffs. If you can apply these two "methodologies" you can
easily have an optimal Git history where you can easily regain context. And this
is one of the powers of Git. 
