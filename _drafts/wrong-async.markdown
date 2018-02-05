---

layout:  post
title: Your async is broken


---
So async. I think most .NET developer have heard anything about `async/await` in C#. Some of them
have used it. And some of some have used it correctly. Amazing, but even now lot of guys misuse
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

// ...

class Service {
  void IWantAsync(){
    new Worker().DoWork().Wait();
  }
}

{% endhighlight %}

In this naive form the code looks meaningless, but it has the pattern I would like to discuss. Usually
authors of code like that have this reasoning: "Well, sometime in the future we'll refactor our `Worker`
class making it true async. And make `Service` async as well. But for now we'll just add some async API."

And here comes the problem. Because code like that, where you have `Wait()` method called, has worse
performance than synchronous version. OK, but why?

To answer this question one may want to recall how async work. Everyone knows that it creates a `Task` behind
the scenes. The `Task` usually is executed in another thread, taken from the `ThreadPool`. The thread 


Another thing: 
await Task.Run(...; return xxx;)
vs
retrun Task.FromResult(...; return xxx;)
