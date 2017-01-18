---

layout: post
title: "Using ELK for Logging on Windows: Configuration"
#date: 2017-01-18 11:00 +02:00
categories: ELK devops windows .NET

---



also

## Logstash


## Adding Firewall Rules


{% highlight cmd %}

netsh advfirewall firewall add rule name="Allow incoming Elasticsearch" protocol=tcp localport=9200 dir=in action=allow
netsh advfirewall firewall add rule name="Allow incoming Logstash" protocol=tcp localport=5044 dir=in action=allow
netsh advfirewall firewall add rule name="Allow incoming Kibana" protocol=tcp localport=5601 dir=in action=allow


{% endhighlight %}

## Filebeat

{% highlight cmd %}
cinst filebeat -y

{% endhighlight %}

Then configure it:
notepad C:\ProgramData\chocolatey\lib\filebeat\tools\filebeat-1.2.3-windows\filebeat.yml

(remember to do _cinst notepad2-mod -y_ to improve your Notepad experience)

The most common settings you'd need to change are:

 - path to your logs
 - destination (logstash)


 ## Grok Patterns

 2016-12-26 12:12:52.320 [ 6] [Debug] Poll PccEmailNotification [Insight4.JobExecutor] 

 ^%{TIMESTAMP_ISO8601:timestamp}\s+\[\s?%{NUMBER:threadid}\]\s+(\[%{WORD:loglevel}\]\s+)?%{GREEDYDATA:message}


 http://grokconstructor.appspot.com/do/match#result