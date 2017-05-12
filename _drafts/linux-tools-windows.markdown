---

layout: post
title: Linux Tools For Windows

---

Steps: 
1. Enable dev mode
2. Install tools via Enable Windows Features
3. Start bash (not git bash). It will prompt to install Ubuntu, confirm it by 'Yes'

Install [linux brew](http://linuxbrew.sh/). You might need to install ruby first. Its easy: `sudo apt-get install ruby`. Steps to install linuxbrew are:
Paste at a Terminal prompt:
{% highlight bash %}
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install)"
PATH="$HOME/.linuxbrew/bin:$PATH"
{% endhighlight %}

Edit your ~/.bash_profile to add ~/.linuxbrew/bin to your PATH:
{% highlight bash %}
echo 'export PATH="$HOME/.linuxbrew/bin:$PATH"' >>~/.bash_profile
{% endhighlight %}

{% highlight bash %}
 . ~/.bash_profile
{% endhighlight %}

You're done! Try installing a package:
{% highlight bash %}
brew install hello 
{% endhighlight %}