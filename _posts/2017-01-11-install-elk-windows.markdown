---
layout: post
title:  "Using ELK for Logging on Windows: Installation"
date:   2017-01-11 17:55:31 +0200
categories: ELK devops windows .NET
comments: true

---

Usually no-one cares about logs in their applications until production. Only when
something bad happens one realizes that prod does not allow to debug and, like, "Where
are our logs?" And then -- "Why don't our logs contain anything useful?". In this post
we'll start with the first question. 

By default logs are written to text files on a local disk. So, to see what is in logs
you need to open it on the remote computer. If you're working with .NET I bet that 
the remote computer is Windows server and you'll need 
[RDP](http://www.it.cornell.edu/services/managed_servers/howto/rdp.cfm). It has UI
and the only tool for reading text files there will be Notepad. It provides 
far from excellent experience though. You can improve it installing 
[Notepad2](http://www.flos-freeware.ch/notepad2.html) or, what is even better, 
[Notepad2-mod](https://xhmikosr.github.io/notepad2-mod/). But still -- connecting to 
remote desktop is something too complicated. 

Next level of comfort is adding a network share to the folder where logs are put.
You could use some utility like [Tailblazer](https://github.com/RolandPheasant/TailBlazer)
to get auto-scroll and coloring and search. But still it does not scale well. If you
have several environments, load balancing, need to search through past events -- 
file shares are bad choice. What can we use instead? Centralized distributed logging.

There are a lot of options in this field, each deserving a separate blog post. For 
now we will see how to use [ELK stack](https://www.elastic.co/products). 

Ok, so I decided to install [ELK](https://www.elastic.co/products) to some fresh 
windows machine (we're .NET guys, remember?). It should be simple as 1-2-3?

1. Install Elasticsearch
2. Install Kibana
3. Install logstash 

However, as usually in our industry, the devil is in the detail. Firstly, there 
are no MSIs for any of elastic products. Secondly, some parts require another 
stuff to be installed. Like, elasticsearch wants JRE, kibana wants JDE and has no 
windows service support out of the box. Hardly I want to log in to that box to 
start Kibana every time I need it (that box is a server after all!). Thankfully, 
there is handy utility, named [Non Sucking Service Manager](https://nssm.cc/) 
(this is real name of the product). It allows to run any executable as a service 
and really rocks. But.. it also comes without installer -- what a nightmare for 
typical Windows user.

I could take that hard and unpleasant way of downloading stuff, putting it to
some folders, making configuration changes and running all sort of bat files... 
To be honest things would not be much better if all those mentioned stuff was 
packaged to MSIs because it would be still *download / run / next / next /... / finish*
experience. This is what makes lives of Windows guys hard.

Why I said "Windows guys"? Because Linux guys haven't ever heard about such problems. 
They got something called package managers handling all installation plumbing work 
for them. You probably noticed that installation instructions for Linux usually compact 
to few lines of bash script that you can just paste into the terminal 
([but be careful](https://www.reddit.com/r/netsec/comments/1bv359/dont_copypaste_from_website_to_terminal_demo/))
.While instructions for Windows versions are full of installer screenshots.

And now it's time for good news. In fact **there are** package managers for Windows.
They are not official (and that's why most of Windows users are not aware of them),
but they do they job and do it well. My favorite is [chocolatey](https://chocolatey.org/). 
It's allows to install software using a single line in console. Moreover, chocolatey itself 
is being installed by a single console line. 

Basically, whats it's doing, is just downloading of installation packages from vendors' 
sites and running them in silent mode. This is dead simple and allows to install 90%
of stuff. You can go to their package listing and find your favorite tools. And next time
when you reinstall Windows, or get new machine, just try installing your stuff via 
chocolatey. You will be impressed how many hours of your life you've wasted clicking **next**
buttons in installers.

So, back to this post title. How to install ELK? You just need to type:

{% highlight powershell %}
cinst elasticsearch -y
{% endhighlight %}

At the moment of this writing elasticsearrch package is not smart enough to create 
and start windows service, so you'd might need to do it manually:

{% highlight powershell %}
pushd C:\\ProgramData\chocolatey\lib\elasticsearch\tools\elasticsearch-2.3.1\bin
service.bat install 
service.bat start
{% endhighlight %}

It also might turn out that service doesn't start because of mess with **JAVA_HOME**. In 
short -- it should be set on machine level but JRE8 intaller sets it only on user lever.
To [solve this](https://chocolatey.org/packages/elasticsearch#comment-2682099756) you need 
to copy that variable to appropriate level. 

And the next step is:

{% highlight powershell %}
cinst kibana -y
{% endhighlight %}

For [some reason](https://disqus.com/home/discussion/chocolatey/chocolatey_gallery_kibana_450/#comment-3142843787) 
kibana service is installed with manual startup type. So after server restart
it will be stopped. However it can be easily fixed:

{% highlight powershell %}
set-service kibana-service -StartupType Automatic
{% endhighlight %}

BTW to quickly check what is the startup type of the service, or other info, you can use
[NSSM](https://nssm.cc/):

{% highlight powershell %}
nssm edit kibana-service
{% endhighlight %}

Another non-obvious thing is that by default Kibana (and Elasticsearch) are bound to `localhost`
that prevents connections from another host. To fix that you need to make changes in `kibana.yml`:

{% highlight yaml %}
server.host: "0.0.0.0"
{% endhighlight %}

And that's all! Behind the scenes it will download zips, unpack it, run registration scripts,
install services and [so on]({{ site.url }}/assets/elastic-install-log.png). Moreover, it detects dependencies and install them if needed. So
we don't need to install JRE, JDE, NSSM explicitly. This is exactly how installation of things 
should be handled in 21st century. Less efforts -- more result. I really like it.

In the next post we'll set up ASP.NET Core logging to target ELK. But first we need to define [additional moving parts](elk-filebeat-logstash) in out setup.

Happy installing!
