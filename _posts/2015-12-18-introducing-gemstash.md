---
layout: post
title: "Introducing: Gemstash"
tags: [ruby, gemstash, bundler]
---

A month ago, the Bundler core team released Gemstash. Let's see what's it's
purpose and how we can leverage it in our everyday jobs.

## Gemstash

From the Gemstash [readme](https://github.com/bundler/gemstash/blob/master/README.md):

> Gemstash is both a cache for remote servers such as https://www.rubygems.org,
> and a private gem source.

This means that, if you often pull gems, Gemstash can cache your gems so you
don't need to fetch them again from the server (i.e. rubygems.org). Also, it has
another purpose - a private gem source. For example, if you have some private
gems, that you wouldn't like to host on a public server, Gemstash can also be a
private gem server.

Let's see how we can configure Gemstash and reap it's benefits.

## Caching

## Private Server

- What is Gemstash and how to use it
    - https://github.com/bundler/gemstash
    - https://github.com/bundler/gemstash/blob/master/docs/private_gems.md
    - https://github.com/bundler/gemstash/blob/master/docs/multiple_sources.md

