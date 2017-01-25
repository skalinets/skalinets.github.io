---
layout: post
title:  "Jekyll, Windows, Pain"
date:   2017-01-10 12:55:31 +0200
categories: .NET fsharp
---

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



