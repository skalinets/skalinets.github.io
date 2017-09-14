---
layout: post
title: Show Me Your Generic Part 2
#date: 2017-05-12 14:30:31 +0200

---

Only few hours passed after I had posted a post about misusing generics when I spotted
a real gem during the code review. Here it is:

{% highlight csharp%}

public class UltraGenericService<T> : BaseGenericService<T>
{
    public T DoSomeStuff<TFoo, TBoo>(TFoo foo, TBoo boo)
        where TFoo : IFoo
        where TBoo : IBoo
    {
        foo.DoFooStuff();
        boo.DoBooStuff();
        return GetTFromSomeWhere();
    }
    ...
}
{% endhighlight %} 

Cool, right? Here we have all sort of coolness: class level generics, method level generics and even generic constraints. However, this code can be simplified. Just check this out:

{% highlight csharp%}

    public T DoSomeStuff(IFoo foo, IBoo boo)
    {
        foo.DoFooStuff();
        boo.DoBooStuff();
        return GetTFromSomeWhere();
    }

{% endhighlight %}  

Yes, exactly. We don't need method level generics here, because, you know, they only intention was to restrict an argument to be a subtype of some interface. And this can be achieved by just using proper signature. Note that as [Sergey Teplyakov](https://disqus.com/by/sergeyteplyakov/) mentioned in comment below, the latter makes
sense if implementations of `IFoo` and `IBoo` structs. But in our particular case those were not (and I don't remember seeing structs being used in such a way).  

Well, code does not longer look very smart, but good code should be simple, and not always smart.

Enjoy your coding :)