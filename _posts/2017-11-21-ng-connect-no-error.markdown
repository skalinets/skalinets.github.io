---

layout: post
title: "connect: No error When Running ng new "
date: 2017-11-21 11:00 +02:00
categories: angular, nodejs

---


It's (almost end of) 2017 and I am ready to jump into "future". This time future is [angular.io](angular.io). BTW,
many call it Angular 2 or Angular 4 to distinguish it from *old* Angular 1.x. That is incorrect, or, to be more
precise, does not scale well. New versions are being released too often and it's rather stupid to replace all `Angular 2`
to `Angular X` every time. More safe and correct is to call new angular **Angular.io** and old one **AngularJS**.

So, I started diving in **Angular.io** with it's [tutorial](https://angular.io/tutorial/toh-pt0). I remember that great
experience with awesome **AngularJA** [Phonecat tutorial](https://docs.angularjs.org/tutorial/step_00) and have strong
feeling that new one should be good as well. With no fear I did that:

{% highlight bash %}
npm -i angular-cli
ng new my-super-app
{% endhighlight %} 

And then fun began. First of all it said that `angular-cli` is (or will be deprecated) and it is going to be (or has
been) renamed to `angular/cli`. It seemed to get installed though. However, `ng new` command just printed **connected: no error** 
and did nothing.

I was not surprised. Well, every time when I start doing something new with node, things like that happens. Node is not
windows friendly after all. I began googling but found nothing related. Sigh. Then I tried to install brand new `angular/cli`
but it refused to install, producing lot of error text messages. I updated node, refreshed my SSH key on github (yes, it is
used during `npm install` process) but with no much luck.

My next thought was "ok, maybe `angular-cli` conflicts with `angular/cli` (do you feel the drama? :)". Let me check where
it got installed. How can I do that? In powershell there is a nice command `get-command`. And how surprised I was when it
printed ` C:\ProgramData\chocolatey\bin\ng.exe`. 

I definitely did not install `angular-cli` via [chocolatey](https://chocolatey.org/). So it was something else. Quick try 
with different help switches ended with `ng --help` saying that this is a  [nailgun](https://github.com/facebook/nailgun)
by facebook. I don't remember when I installed it. I did not also find it in [chocolatey repository](https://chocolatey.org/packages?q=nailgun).
Having no idea where did it appear on my laptop I just did

{% highlight powershell %}
pushd C:\ProgramData\chocolatey\bin\
Rename-Item .\ng.exe ng-fb.exe
popd
{% endhighlight %}

And got my angular-cli back. Well, I'd say it was very small sacrifice to node / npm / angular, and nothing will stop
me from getting that tutorial done!