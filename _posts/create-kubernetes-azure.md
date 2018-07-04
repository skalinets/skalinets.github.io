---
title: Creating Kubernetes Cluster in Azure with PowerShell
---

To start with that we need to make sure that our Azure PowerShell is up to date. To do it
we can run following command under administrator:

{% highlight powershell %}
Install-Module -Force AzureRM
install-module -Name azurerm.aks -AllowPrerelease
{% endhighlight %}


{% highlight powershell %}
Select-AzureRmSubscription 'some subscription name'
$rg = 'aks-playground' # save for later use
New-AzureRmResourceGroup -Location 'west europe' -name $rg

{% endhighlight %}

You might also need to setup Kubernetes CLI: 

{% highlight powershell %}
choco install kubernetes-cli
{% endhighlight %}

There was [annoying bug](https://github.com/kubernetes/kubernetes/issues/65575), so you might need to 
downgrade the version of `kubectl`:


{% highlight powershell %}
choco upgrade kubernetes-cli --version 1.10.2 -y --allow-downgrade
{% endhighlight %}



{% highlight powershell %}
choco upgrade kubkubectl get nodes
NAME                     STATUS    ROLES     AGE       VERSION
aks-default-10789655-0   Ready     agent     26m       v1.8.1
aks-default-10789655-1   Ready     agent     26m       v1.8.1
aks-default-10789655-2   Ready     agent     26m       v1.8.1
{% endhighlight %}