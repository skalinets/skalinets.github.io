---
title: Docker for Bums 
---

I have a rather [decent laptop](https://www.notebookcheck.net/Razer-Blade-14-Early-2015-Notebook-Review.138997.0.html) with 
i7 and SSD and all that stuff, but with Windows Home preinstalled. For some unknown reasons it cannot be updated to
Windows Pro (at least in easy way, and if it could, it would costs $100), so, yeah -- Windows Home. It's not that bad, 
I can run anything I need for my dev work, however there is one gem of software enginery that cannot run there and it is
a [Docker For Windows](https://www.docker.com/products/docker-desktop). 

It cannot be installed because it requires a HyperV that is available only with Pro version of Windows. However, there is a 
workaround. It is called a "legacy desktop solution", but it works rather nice. It is a [Docker Toolbox](https://docs.docker.com/toolbox/).
And in this post I will share my setup. Just in case you don't have a "normal" Windows :)

Bash
==

After installation you get a **Docker Quick Start Terminal** in your start menu. It works 
pretty nice, starts a virtual machine if it hasn't been started and allows you to do all docker stuff.
It's good, but I don't like the default bash console. It has ugly fonts, no [quake](https://conemu.github.io/en/SettingsQuake.html) 
effect and sucks in general. I'd like to use it in my gorgeous [Conemu](https://conemu.github.io/). 

To add it to Conemu, you just need to open Conemu's Settings/Tasks and click button **Add/refresh default tasks...**

Powershell
==

It works in Bash, and Bash is cool, but I prefer Powershell. However, when I tried to run docker from powershell, I got this:

```
error during connect: Get http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.33/images/json:
open //./pipe/docker_engine: The system cannot find the file specified. In the default 
daemon configuration on Windows, the docker client must be run elevated to connect. 
This error may also indicate that the docker daemon is not running.
```

First time when I saw it, I decided to go with Bash. You need to learn bash anyway if you want to work with Docker.
But it was not very convenient -- I was switching between tabs, from Powershell to bash and back. So eventually I
decided to make it work in Powershell as well.

It turned out to be very easy thing. You just need to configure your docker client (basically `docker`) to connect to
your virtual machine. It could be easily done in bash, you just need to run `$ eval "$(docker-machine env default)"`.
Obviously it does not work in Powershell. But we're hackers, aren't we?

I tried to run `docker-machine env default` in bash (without eval) and got the following:

{% highlight bash %}
$ docker-machine env default
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="C:\Users\admin\.docker\machine\machines\default"
export DOCKER_MACHINE_NAME="default"
export COMPOSE_CONVERT_WINDOWS_PATHS="true"
# Run this command to configure your shell:
# eval $("C:\Program Files\Docker Toolbox\docker-machine.exe" env default)

{% endhighlight %} 

It is just a bunch of environment variables, that also can be easily set up with powershell:

{% highlight powershell %}
$env:DOCKER_TLS_VERIFY="1"
$env:DOCKER_HOST="tcp://192.168.99.100:2376"
$env:DOCKER_CERT_PATH="C:\Users\admin\.docker\machine\machines\default"
$env:DOCKER_MACHINE_NAME="default"
$env:COMPOSE_CONVERT_WINDOWS_PATHS="true"
{% endhighlight %} 

Once that command is run, `docker` starts working. Very good!

Other tools
==

So we've got it working with powershell. You still need to run that script in every powershell console, but it could 
be easily solved by adding it to the powershell profile. However, there is other issue.

You might have other tools, that rely on docker. For instance, I use Visual Studio Code and there are bunch of plugins
for Docker. They don't work, because still cannot connect to Docker daemon. How to fix them?

Easy. Just run the following code as Administrator:

{% highlight powershell %}
[System.Environment]::SetEnvironmentVariable("DOCKER_TLS_VERIFY", "1", "Machine");
[System.Environment]::SetEnvironmentVariable("DOCKER_HOST", "tcp://192.168.99.100:2376", "Machine");
[System.Environment]::SetEnvironmentVariable("DOCKER_CERT_PATH", "C:\Users\admin\.docker\machine\machines\default", "Machine");
[System.Environment]::SetEnvironmentVariable("DOCKER_MACHINE_NAME", "default", "Machine");
[System.Environment]::SetEnvironmentVariable("COMPOSE_CONVERT_WINDOWS_PATHS", "true", "Machine");
{% endhighlight %} 

It does the same stuff as previous snippet, however environment variables are set in machine scope and are available 
to all new processes. 

Enjoy!