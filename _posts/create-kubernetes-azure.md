---
title: Creating Kubernetes Cluster in Azure from console
---



To start with that we need to make sure that our Azure PowerShell is up to date. To do it
we can run following command under administrator:

{% highlight powershell %}
Install-Module -Force AzureRM
install-module -Name azurerm.aks -AllowPrerelease
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

1. Create resource group
{% highlight powershell %}
$rgname = 'your_resource_group'
New-AzureRmResourceGroup -Name $rgname -Location 'westeurope'
{% endhighlight %}

2. Create AKS cluster
{% highlight powershell %}
$cluster = 'my_awesome_cluster'
az aks create --resource-group $rgname --name $cluster
{% endhighlight %}

3. Connect to cluster using kubectl
{% highlight powershell %}
az aks get-credentials --resource-group $rgname --name $cluster
kubectl get nodes
{% endhighlight %}

4. Create Azure Disk:
{% highlight powershell %}
az disk create --resource-group $rgname `
 --name msAKSdisk  --size-gb 10  --query id --output tsv 
{% endhighlight %}

