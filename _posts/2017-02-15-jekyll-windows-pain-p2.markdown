---

layout: post
date:   2017-02-15 16:25:31 +0200
title:  "Jekyll, Windows, Pain: Part 2"
categories: Jekyll Disqus 

---

_See the first part [here](jekyll-windows-pain-p1)._

In the previous [post](jekyll-windows-pain-p1) we discussed Jekyll and Pain, but there was no Windows.
Now we'll fix that.

Let's rewind the time and get back to the moment of Jekyll installation. There are good chances that
instead of happy path poor windows guy will stuck with the following:

{% highlight powershell %}
gem install jekyll bundler
ERROR:  Could not find a valid gem 'jekyll' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)
ERROR:  Could not find a valid gem 'bundler' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)
{% endhighlight %}

Thanks to gods it can be easily _googled out_ and fixed ([here](https://gist.github.com/luislavena/f064211759ee0f806c88) 
is an instruction from old times and in Dec 2016 they put it to [official docs](http://guides.rubygems.org/ssl-certificate-update/#installing-using-update-packages)).
There is a long [discussion and description of the issue](https://github.com/rubygems/rubygems/issues/1050#issuecomment-61422934).
Someone even says that not only Windows systems are affected, but I noticed that issue only with Windows. 
And I still don't understand why it cannot be fixed with installer (I was just impatient to read the 
whole thread) :) But anyway...

Once I installed Jekyll and decided to use built-in `minima`'s Disqus integration I figured out
that Disqus is broken when published on Github. It was just saiyng that something is bad with configuration,
despite the fact that I've followed `minima`'s instructions precisely. After investigation it turned out
that `minima` introduced some changes related to how scripts are rendered on the page. And that updates
had not been promoted to Github pages when I published my posts. Because Github pages use some particular
versions and not ones that comes with your `Gemfile`.

Here comes the need to match Github version of `minima`. Thanks to the internet the solution is easy.
Instead of specifying the version of `minima` and other components manually you can replace all them
with a single gem [github-pages](https://github.com/github/pages-gem). It has dependencies to versions
`minima` and other gems that most closely match ones running on github. 

With that good knowledge I updated my `Gemfile`, pushed blog and finally got Disqus working!
Hooray! But...

When I tried to see my blog locally (after `jekyll serve`) I received some weird error:

{% highlight cmd %}
`connect': SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (OpenSSL::SSL::SSLError)
{% endhighlight %}

Again googling found [this](https://github.com/jekyll/jekyll/issues/3985) and [this](https://github.com/jekyll/jekyll-gist/issues/30)
and finally a [solution](https://gist.github.com/fnichol/867550). In short: one of `github-pages`'s 
dependencies, [jekyll-gis](https://github.com/jekyll/jekyll-gist), despite the fact that I don't use it, 
makes https requests to gist.github.com.  
And, as you might guess now, gets messed up with SSL certificates. The fix just updates the list of trusted 
certificates. And this error again is Windows specific.

This short blog post does not reflect the level of pain that regular windows guy might feel when trying
to deal with Ruby. Ridiculous that they not have been fixed yet. But there are also the good news. Everything 
seems to have workarounds (some of them are pretty old). Just google and have fun instead of pain. 

PS. One of the comments to the first post of these series pointed out that the easiest way would be
running jekyll inside local docker container. And I should agree with it. Of course that seems to be too easy,
but maybe I will give it a try some day.  
