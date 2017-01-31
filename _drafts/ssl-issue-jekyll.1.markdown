---
layout: post
title:  "Jekyll, Windows, Pain: Part 1"
date:   2017-01-10 12:55:31 +0200
categories: .NET fsharp
---


As you might have noticed I am using [Jekyll](http://jekyllrb.com/) for this blog. It is
really nice and allows you to generate content very quickly. Being developer I am using
my everyday tools for blogging: Visual Studio Code and git. Something like described
[here](http://neoelemento.com/blog/2016/04/25/my-blogging-workflow/) (but not exactly).

Setting up Jekyll on Windows is not that hard, expecially if you're using cool stuff like
[chocolatey](https://chocolatey.org/). It's just about 
{% highlight powershell %}
cinst ruby -y
gem install jekyll bundler
\# ... and the rest stuff from http://jekyllrb.com/
{% endhighlight %}

Just too easy, right? 

Next step is adding ability to comment. Unlike guys requiring to make [pull request](https://github.com/ploeh/ploeh.github.com#comments) with comments I prefer [Disqus](https://disqus.com/). It is [extremely easy](https://tdd4net.disqus.com/admin/universalcode/) to get installed to the blog. I was really quick in the first step of ["Make It Work Make It Right Make It Fast"](http://wiki.c2.com/?MakeItWorkMakeItRightMakeItFast) workflow: adding to the single post. But when I tried to proceed to the next level and add that to the post template,
I figured out that there is no post template at all in my site folder. It turned out
that default stuff like templates, styles and so on [is not kept](https://github.com/jekyll/minima/issues/38) in the blog directory. But rather in some centralized location. After quick googling I found magic spell that reveals that location theme ([minima](https://github.com/jekyll/minima) in my case) used by Jekyll:

{% highlight powershell%}
bundle show minima
{% endhighlight %}

So I copied templates from there to my local site and started to add Disqus code there. When I was almost done I noticed that Disqus is supported [out of the box](https://github.com/jekyll/minima#enabling-comments-via-disqus). Just like [Google Analytics](https://github.com/jekyll/minima#enabling-google-analytics) that I also had hand-crafted before seing that Readme.md.

Well, I realized that I just had done things in exactly opposite way. Instead or
reading the manual and changing several lines in configuration I spent few hours copying
and pasting JS code then trying to recreate default page templates then looking for
location of default templates etc...

Finally I got OK. I reverted all my changes made needed configuration changes and even got Disqus comments showing on my local machine... But I was far from the happy
end. 

_to be continued..._

[minima]

When 

Error:

{% highlight cmd %}
   GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
jekyll 3.3.1 | Error:  SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed
{% endhighlight %}


from [here](https://github.com/jekyll/jekyll-gist/issues/30) to 
[here](https://github.com/oneclick/rubyinstaller/issues/313) to 
[here](https://gist.github.com/fnichol/867550#the-manual-way-boring)

The result is that you need to download [this file](https://curl.haxx.se/ca/cacert.pem) and point
`SSL_CERT_FILE` environment variable  to it.



