---
layout: post
title: "Using multiple sources in a Gemfile"
tags: [angularjs, google, javascript]
---

Gemfiles require at least one gem source, in the form of the URL for a RubyGems server. Although it's not recommended, it's possible as of Bundler 1.7, to add multiple global `source` lines. Each of these sources has to be a valid Rubygems repository.

When using multiple sources, bundler shows a warning message:

![1\. ie@Iles-MacBook-Air: ~/dev/siyelo/vpn_pricing (zsh) 2015-04-15 02-09-01.png](https://api.monosnap.com/rpc/file/download?id=IEECDKIvujGgdHqA1cguNQE1pRBPs7 "1\. ie@Iles-MacBook-Air: ~/dev/siyelo/vpn_pricing (zsh) 2015-04-15 02-09-01.png")Although, this warning can be disabled by running the

**bundle config disable_multisource true**

command, there's a better approach to this.

### :source

Just like one can use the **:github**, to fetch a gem from a Github repo, there's also an option that allows the selection of an alternate Rubygems repository for a gem using the ':source' option.

{% highlight ruby %}
gem "my_gem", :source => "https://mygems.com"
{% endhighlight %}

This forces the gem to be loaded from the specified source and ignores any global sources declared at the top level of the file. If the gem does not exist in this source, it will not be installed.

### **Source blocks**

Just like **group **accepts a name and a block as arguments, there's a **git** and **source** blocks that allow one to specify the source of the gems (s)he want's installed.

{% highlight ruby %}
source "https://my.rubygems.server.com" do
  gem "my_api_gem"
  gem "my_api_logging_gem"
end
{% endhighlight %}

This tells Bundler to fetch my gems from my Rubygems server. If he can't find them on that server, it does not fall back to any other sources specified in the Gemfile. Also, there's the **git** block. This is an example from the Bundler documentation.

{% highlight ruby %}
git "https://github.com/rails/rails.git" do
  gem "activesupport"
  gem "actionpack"
end
{% endhighlight %}

Since the Rails Github repo contains all of the gems that consist the Ruby on Rails framework, the repo can be used as a kind of a source server. This will tell Bundler to look for these gems in this git repo and install them.
