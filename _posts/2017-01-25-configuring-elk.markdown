---

layout: post
title: "Using ELK for Logging on Windows: Configuration"
date: 2017-01-25 12:00 +02:00
categories: ELK devops windows .NET

---


_This is the 3rd post about setting up distributed logging with ELK stack. Previous are [here](install-elk-windows)
and [here](elk-filebeat-logstash)._

So far we discussed our approach regarding adding distributed logging to an application. Our case mostly covers setups
where there is already default filebased logging and we'd like to add centralized processing for them, without much
configuration / code changes. But it also could be used to setup up logging from scratch.

As mentioned [here](elk-filebeat-logstash), to ship log files to Elasticsearch, we need Logstash and Filebeat. Let's get them installed.

### Filebeat

Filebeat should be installed on server where logs are being produced. In install it we'll use
[chocolatey](https://chocolatey.org/packages/filebeat):

{% highlight shell %}
cinst filebeat -y -version 5.1.2

{% endhighlight %}

>Note that we explicitly specified version here since at the moment of this writting some very old version is default
in Chocolatey. And it contains weird bugs. Chocolater has the process of moderation and it takes some time for newest versions to get approved. Thus one more advice: always check the version that gets installed.

Then configure it via 

{% highlight cmd %}
notepad C:\ProgramData\chocolatey\lib\filebeat\tools\filebeat-1.2.3-windows\filebeat.yml

{% endhighlight %}


_remember to do `cinst notepad2-mod -y` to improve your Notepad experience_

The most common settings you'd need to change are:

 - path to your logs
 - destination (logstash)

Here is our configuration:

{% highlight yml %}
filebeat:
  prospectors:
    -
      paths:
         -  C:\Octopus\Applications\Dev2\*\*\Logs\*.txt
      encoding: utf-8
      input_type: log
      multiline:
        pattern: '[0-9]{4}-[0-9]{2}-[0-9]{2}'
        negate: true
        match: after
  registry_file: "C:/ProgramData/filebeat/registry"
output:
  logstash:
    hosts: ["ourserver:5044"]
logging:
  to_files: true
  files:
    path:  C:\ProgramData\chocolatey\lib\filebeat\tools\filebeat-5.1.2-windows-x86_64\mybeat
    rotateeverybytes: 10485760 # = 10MB
  level: info
{% endhighlight %}


Highlights:

- `paths` may contain asterisks
-  `multiline` should be set to treat multiline log entries as a single one. By default every 
line will be a separate entry. Here we define `pattern` as a date that is placed at the
beginning of every line and combination of `negate` and `match` means that every line, not
started with `pattern` should be appended to the previous message.
- `logging` is enabled and `path` is set explicitly as absolute. This is because when
filebeat is started as a windows server, it's home directory is (surprise!) `%WINDIR%\System32\`.
- `registry_file` is a file containing information about current status of filebeat. Among
other, it has last processed positions for all files being monitored by filebeat. So, to
re-process files again all you need is to delete that file and restart filebeat. Or, to be
more precise: 

{% highlight powershell %}

stop-process filebeat #or stop-service filebeat
del C:/ProgramData/filebeat/registry 
start-process filebeat #or start-service filebeat

{% endhighlight %}

Usually its a good idea to check configuration by running console version of application and once it is stable and working -- install and run service. This approach save tons of time.


### Logstash

First question: where should it be installed? Common scenario is to install it on the same machine as Elasticsearch.
However it might be on different machine. This might be useful in setups where a log of log data is expected and Logstash
becomes a bottleneck. You can have more than one installed as well.

We'll use [chocolatey](https://chocolatey.org/packages/logstash) here as usual:

{% highlight cmd %}
choco install logstash
{% endhighlight %}

This command installs Logstash and creates a service. Service is stopped by default and you should start it manually.
This is done of good reason: in 99.(9)% cases further configuration is needed. Configuration is stored in `logstash.yaml`
(default location is `C:\logstash\conf.d\logstash.conf`).

In our case we need to teach it to parse our text messages that will come from Filebeat. Example of message is:

{% highlight cmd %}
2016-12-26 12:12:52.320 [ 6] [Debug] Here is some very important message 
{% endhighlight %}

We will parse it with such filter:

{% highlight cmd %}
filter {
  grok {
    match => { "message" => "^(\xEF\xBB\xBF)?%{TIMESTAMP_ISO8601:timestamp}\s+\[\s?%{NUMBER:threadid}\]\s+(\[%{WORD:loglevel}\]\s+)?%{GREEDYDATA:message}$" }
    overwrite => [ "message" ]
    }
  grok {
    match => { "source" => "\\(?<environment>(\w|\d)+)\\Insight4\.(?<service>[^\.]+)[^\\]*\\(?<version>[^\\]+)" }
  }
  date {
    match => [ "timestamp", "yyyy-MM-dd HH:mm:ss.SSS"]
    timezone => "US/Eastern"
    remove_field => ["timestamp"]
  }
}

{% endhighlight %}

Configuration language of Logstash seems to be a bit cryptic, but it starts to make sense after a couple of hours (or days) spent
with it. Short translation of above snippet:

- There are three parts on the filter
- The first part uses [Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) language to parse a
string and extract values from it. Grok is built on top of regular expression and is not that ugly as it might seem to be.
For instance, our first part is:
  - `^` -- start of the string
  - `(\xEF\xBB\xBF)?` -- optional [BOM](https://en.wikipedia.org/wiki/Byte_order_mark) at the start of the string (more about that is below)
  - `%{TIMESTAMP_ISO8601:timestamp}` -- matches [timestamp](https://github.com/hpcugent/logstash-patterns/blob/master/files/grok-patterns#L72)
pattern and saves it as `timestamp` field
  - `\s+\[\s` -- one or many whitespaces, followed by opening square bracket, followed by a single whitespaces
  - `%{NUMBER:threadid}\]` -- matches number and saves it to `threadid`... and so on
- `overwrite` instructs Logstash to replace current value of `message` field with results extracted with `match`. 
The result is that `message` field will not contain date, log level and so on. 
- Next grok template extracts environment specific data from `source` field. That field in our 
case contains path to log file and our logs are stored in specific place and we can use parts 
of that path to get environment name, version and a service name.
- Finally we instruct logstash to fix the entry timestamp. By default the date or every 
entry is when it was processed by Filebeat. Using `date` filter we extracts correct value from the log entry. And we also explicitly set our time zone. Because for some reason our logs use
local time and you might be surprised when trying to find needed items from browser running in
another time zone.   

As we can see it is not that complicated. And you can always test your patterns on [grokconstructor](http://grokconstructor.appspot.com/do/match).
Such tools are extremely helpful for troubleshooting. Regular expressions are simple, but it is sometimes difficult to write
them correct from scratch. In most cases they need to be tuned. And online testers come to help. Remember BOM symbols at the
begining of my above grok sample? There was a good reason to add them. 

The story is that. I've configured filebeat and logstash on one server and copied configuration
 to another one. But it didn't work there. I copied grok pattern to [grokconstructor](http://grokconstructor.appspot.com/do/match) as well as log samples from
both servers. And figured out that log records for latter server was [not matched](http://i.imgur.com/Sjayk34.png). After changing test
entries back and forth I understood that something invisible at the beginning of the string was preventing my beautiful pattern
from matching those lines. And online hex editor [detected them](http://i.imgur.com/XdOFcTA.png). Then I added optional BOM to the start of my pattern and it started to recongnize all
entries. I am not sure why logs from one server have them and from other one does not, but we
have a fix and it works for us.

Once config is set, we start our service:

{% highlight powershell %}
start-service logstash
{% endhighlight %}

### Adding Firewall Rules

One important thing is to enable inbound connections to our servers. This can be done via 
[Windows Firewall UI](http://www.howtogeek.com/112564/how-to-create-advanced-firewall-rules-in-the-windows-firewall/)
 or via simple script:

{% highlight cmd %}

netsh advfirewall firewall add rule name="Allow incoming Elasticsearch" protocol=tcp localport=9200 dir=in action=allow
netsh advfirewall firewall add rule name="Allow incoming Logstash" protocol=tcp localport=5044 dir=in action=allow
netsh advfirewall firewall add rule name="Allow incoming Kibana" protocol=tcp localport=5601 dir=in action=allow

{% endhighlight %}

> Consider efforts required by those two approaches (given that you know what buttons should be
clicked and what commands should be typed in console). Console based is much faster. And if you
found solution in internet can copy & paste script from site, while still would need to click through UI, guided by screenshots and instructions. But keep in mind that copied script can be [harmful](https://www.reddit.com/r/netsec/comments/1bv359/dont_copypaste_from_website_to_terminal_demo/).

And that's all! Once all our services are started, we have our distributed logging working.



