---

layout:  post
title: Your async is broken


---
So async. I think most .NET developer have heard anything about `async/await` in C#. Some of them
have used it. And some of some have used it correctly. Amazing, but even now lot of guys mistreat
asyncs completely. 

I spotted the code like this:


{% highlight csharp %}

class Worker {
  async Task DoWork()
  {
    DoSomeThingSyncronously();
    await Task.FromResult(1);
  }
}

class Service {
  void IWantAsync(){
    
  }
}

{% endhighlight %}