One of our current projects is a multitenant system. Recently we've got a requirement to change IDs of
our tenants from hard to guess random cryptic (eg. 'adfx65sd') to natural ones. There are two areas for 
changes: database and filesystem. The database is handled with EF migration rather nicely. 

However filesystem is slightly more complicated. We have those IDs being spread accross different folders, 
i.e. there are tenant specific styles, images and so on. Hot to perform rename in some smart way?

First we need to find items that should be renamed. It is easy to do even with Windows File Explorer (open
the folder with your source files and just search by tenant ID). You can go further and rename stuff just
in search results. But that is... no fun at all. Files and folders have different naming schemas, and we 
have more than one tenant, so this would become boring and error prone.

Remember that we are developers and our passion is to write code. So, lets solve this with code! Our
choice is a Powershell. I posted once how it can be used to [parse files]({{ site.baseurl }}{% post_url 2017-02-02-powershell-regex %}), make sure to
read it, some Powershell mechanics used below is explained there. 

Finding files that should be renamed is rather easy:

{% highlight powershell %}
ls *old_tenant_id* -Recurse
{% endhighlight %}

We should get list of files and folders that are subject to rename. What to do next? 

Powershell provides lot of useful stuff to work with files. Our today's hero is [Rename-Item](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-item?view=powershell-6).
It has two arguments, what should be renamed and what new name should it get. Previous `ls` gives us that
first one. What about other?

If I was using C# I would try `String.Replace()`. I am not sure if Powershell has anything like that, but I don't care.
Remember that Powershell is build on top of .NET. And it means 2 things:
1. Everything is an object.
1. .NET APIs are available.

It means that we can use `Name` property of each file / folder, and since it's type is a `String`, we can
also use `String.Replace()`.

Let's confirm our theory first:

{% highlight powershell %}
ls *old_tenant_id* -Recurse `
  | % { $newName = $_.Name.Replace('old_tenant_id', 'new_tenant_id'); `
        write-host "$_ -> $newName" }
{% endhighlight %}

This should print all files / folders with their new names. If everything is fine, we can proceed with renaming:

{% highlight powershell %}
ls *old_tenant_id* -Recurse `
  | % { 
      Rename-Item -path $_ -NewName $_.Name.Replace('old_tenant_id', 'new_tenant_id') 
  }
{% endhighlight %}

It does the trick! Now all our stuff is renamed and ready to be committed. 

But that is not all. Remember, we have many tenants. Sure, we can go ahead and just update previous script and run it for every tenant. However what if we could do it once for all?

The solution is rather simple. Just add all old and new IDs for each tenant to a dictionary and iterate over it. Here is the script:

{% highlight powershell %}
@{'oldtenant1_key'='newtenant1_key';'oldtenant1_key'='newtenant1_key'} `
  .GetEnumerator() ` # this is needed by %{} 
 | % { 
   $kvp = $_; ` # because $_ will be shadowed in the inner loop
   ls "*$($kvp.Key)*" -Recurse  `
        | % { Rename-Item -path $_ -NewName `
            $_.Name.Replace($kvp.Key, $kvp.Value) -Force }}
{% endhighlight %}

And finally all our assets got renamed by this simple (almost) oneliner. 

