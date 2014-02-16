---
layout: post
title:  "Installing Jekyll on Windows errors"
date:   2014-02-16 14:42:48
categories: jekyll
---

So today I set up this github.io page (easy) and [Jekyll][jekyll]. The latter went not that smooth so this first post will point you to the solution of my issues.

I was trying to use Jekyll on Windows. After installing and uninstalling wrong versions of Ruby, Ruby DevKit and (eventually Portable) Python, the error I got was:


{% highlight bash %}
$ c:/programs/Ruby192/lib/ruby/1.9.1/fileutils.rb:247:in 'mkdir': Invalid argument
- e:/dev/testblog/_site/e: (Errno::EINVAL)
...
{% endhighlight %}
Took some time to figure out the latest version of Jekyll (1.4.3) does not work on Windows and I had to down grade one version:

{% highlight bash %}
gem uninstall jekyll

gem install jekyll --version "=1.4.2"
{% endhighlight %}

Thank you [stackoverflow][jekyll-fix].

When trying to run

{% highlight bash %}
jekyll server --watch
{% endhighlight %}
I got

{% highlight bash %}
$ c:/programs/Ruby192/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in
'require': no such file to load -- wdm (LoadError)
...
{% endhighlight %}

and the solution was to install wdm like:

{% highlight bash %}
gem install wdm
{% endhighlight %}

After this, using Jekyll works as intended.

I started out thinking setting up a nice looking github page would be relatively easy but ended up installing and working around issues way more than expected, resulting in a even more unexpected first blog post.

[jekyll-fix]: http://stackoverflow.com/questions/21137096/jekyll-error-running-jekyll-serve
[jekyll]:    http://jekyllrb.com