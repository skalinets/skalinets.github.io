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

Yes, exactly. We don't need method level generics here, because, you know, they only intention was to restrict an argument to be a subtype of some interface. And this can be achieved by just using proper signature. 

Well, code does not longer look very smart, but good code should be simple, and not always smart. 

Enjoy your coding :) 


Note that this is VERY simplified version, it allows only to get object by ID and
save in the database. The set of operations is irrelevant to this post. The important part here is that we would need to explicitly cast our entities back and forth to and from `object`. To avoid that pain, generics can and should be used.

And here comes the pitfall. I often have seen declarations like that:

{% highlight csharp%}

public interface IRepository<T>
{
    T GetById(int id);
    void Save(T entity);
}

{% endhighlight %} 

It might seem to be completely OK. Implementation usually is also very straightforward:

{% highlight csharp%}
public class Entity {  public int Id { get; set; } }

public class Repository<T> : IRepository<T> where T : Entity
{
    private readonly DbContext context;

    public Repository(DbContext context)
    {
        this.context = context;
    }

    public T GetById(int id)
    {
        return context.Set<T>().Single(_ => _.Id == id);
    }

    public void Save(T entity)
    {
        context.Attach(entity);
        context.SaveChanges();
    }
}

{% endhighlight %} 

Again -- don't blame me for non-working code :) I used EF here as example. The main point is that implementation of this class is generic as well. And this is good -- you don't need to add separate repositories for every of your entities. 

But consider how it's gonna be used. It your service needs to operate with, let's say, `User`, `Account` and `Transaction`, its constructor will be like:

{% highlight csharp %} 

public class MyService
{
    private readonly IRepository<User> userRepository;
    private readonly IRepository<Account> accountRepository;
    private readonly IRepository<Transaction> transactionRepository;

    public MyService(
        IRepository<User> userRepository, 
        IRepository<Account> accountRepository, 
        IRepository<Transaction> transactionRepository)
    {
        this.userRepository = userRepository;
        this.accountRepository = accountRepository;
        this.transactionRepository = transactionRepository;
    }
}

{% endhighlight %}

Do you see it? We've just added 3 dependencies each being the same generic class. Try to count the word `repository` here. Our class still does nothing but has a lot of code already.

Probably we could have designed it better. Check this:

{% highlight csharp %}

public interface IRepository
{
    T GetById<T>(int id) where T : Entity;
    void Save<T>(T entity) where T : Entity;
}

public class Repository : IRepository
{
    private readonly DbContext context;

    public Repository(DbContext context)
    {
        this.context = context;
    }

    public T GetById<T>(int id) where T : Entity
    {
        return context.Set<T>().Single(_ => _.Id == id);
    }

    public void Save<T>(T entity) where T : Entity
    {
        context.Attach<T>(entity);
        context.SaveChanges();
    }
}

public class MyService
{
    private readonly IRepository repository;

    public MyService(IRepository repository)
    {
        this.repository = repository;
    }
}
 
{% endhighlight %}

Yes, the only one dependency has left in our service. It still can work with any of your entities:

{% highlight csharp %}

public void DoWork()
{
    var user = repository.GetById<User>(42);
    var account = repository.GetById<Account>(350);
    ...
    repository.Save(user);
}

{% endhighlight %}
 
[IRL](http://www.urbandictionary.com/define.php?term=IRL) where you have dozens of services and entities it is even more impressive because it saves a lot of useless [LOCs](https://en.wikipedia.org/wiki/Source_lines_of_code). That in turn makes working with your code much more pleasant. It may not work well if you are paid per LOC though :)

This might not work in all cases of course. If your repository has some logic (initialization, caching etc) this pattern might not apply. But still one of your main
goals should be reducing the number of dependencies. This is directly related to [loose coupling](https://en.wikipedia.org/wiki/Loose_coupling) that is considered to 
be a good practice. And good developers strive to follow good practices don't they? :)