---
title: Starting Linkerd Dashboard from Windows System for Linux
---

Linkerd is the first and probably the most famous service mash framework. It adds transparent proxies
to your services in Kubernetes cluster, providing runtime debugging, observability, reliability, and
security — all without requiring any changes to your code. You can read more about it on
[their site](https://linkerd.io/2/overview/) or/and listen to [Software Engeneering Daily Podcast](https://softwareengineeringdaily.com/2018/12/19/linkerd-service-mesh-with-william-morgan/).

One of it's strenghs is an ease of use. And installing should not be difficult as well.

So I opened a [Getting Started page](https://linkerd.io/2/getting-started/index.html) and got it installed
in minutes. However when I tried to open a dashboard, got this error:

{% highlight bash %}
Failed to open Linkerd URL http://127.0.0.1:64538/api/v1/namespaces/linkerd/services/linkerd-web:http/proxy/ 
in the default browser: exec: "xdg-open": executable file not found in $PATH%
{% endhighlight %}


It turned out that for some reason WSL (Windows Subsystem for Linux) does not support `xdg-open` command.
So I needed to do "OK, google" to handle that.

Suprisingly, I needed to do several searches, until found [something useful](https://wpdev.uservoice.com/forums/266908-command-prompt-console-windows-subsystem-for-l/suggestions/15590718-equivalent-of-cgystart-open-or-xdg-open).
So to save one's time I decided to write this post.

Idea is to create a script `xdg-open` that will run `cmd.exe`. To make it available from anywhere it should
be put into PATH. First we check what is in PATH with `echo $PATH`. I have chosen `/usr/local/bin`, created a
file named `xdg-open` there and set it's content to

{% highlight bash %}
#! /bin/sh 
cmd.exe /c "start $*"
{% endhighlight %}

Test it with `xdg-open https://skalinets.github.io`. If you see my blog in new browser window -- it's all good :).

After that `linkerd dashboard` works like a sharm.

Happy hacking!