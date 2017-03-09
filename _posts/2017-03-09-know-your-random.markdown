---

layout: post
title: "Do you know your random?"
date: 2017-03-09 12:00:31 +0200
category: .NET C# "Unit Tests"

---

My previous post contained a bug:

{% highlight csharp %}
public class BlinkingInside
{
    public static readonly IEnumerable<object[]> Foo = Enumerable.Range(1, 1000).Select(i => new object[] { i });

    private readonly int randomValue = new Random().Next(100);

    [Theory]
    [MemberData(nameof(Foo))]
    public void i_am_blinking(int _)
    {
        randomValue.Should().BeLessOrEqualTo(70);
    }
}
{% endhighlight  %} 

Despite the fact that `Random` gets created with every new test run, `randomValue` will be
the same. Why?

To answer this question we might want to get closer to how `Random.Next()` works. I am not going
to [repeat internet](http://softwareengineering.stackexchange.com/questions/109724/how-do-random-number-generators-work),
just _TL;DR;_: random generators generate pseudorandom sequences configured by initial `seed` value.
That said, two instances of `Random` with the same seed will return the same sequences:

{% highlight csharp %}
[Fact]
public void two_randoms()
{
    var random = new Random(42);
    random.Next().Should().Be(1434747710);
    random.Next().Should().Be(302596119);
    random.Next().Should().Be(269548474);
}
{% endhighlight  %} 
 
It can be handful when you need some level of predictability, e.g. you need to reproduce a bug. 
But what's about default constructor `new Random()`? What seed is used in this case? The expectation
is that it should be something _random_ to generate different sequences for different instantiations. But that
does not happen. The default value is rather unfortunate. Consider following test:

{% highlight csharp %}
[Fact]
public void weird_random()
{
    var r1 = new Random();
    var r2 = new Random();
    r1.Next().Should().Be(r2.Next()); // the same

    var r3 = new Random();
    Thread.Sleep(10);
    var r4 = new Random();
    r3.Next().Should().NotBe(r4.Next()); // now not the same
}
{% endhighlight  %} 

`r3` is not equal to `r4` because of `Thread.Sleep`. And everything gets clear after looking at
default `Random` constructor:

{% highlight csharp %}
public Random() 
    : this(Environment.TickCount) {
}
{% endhighlight  %} 

If we create several instances within a single Tick, that lasts [~10-16 milliseconds](https://msdn.microsoft.com/en-us/library/system.environment.tickcount%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396), they would return the same sequences. That behavior may cause hard to reproduce
race condition bugs.

To mitigate that risk we need some more reliable random. And there is one! Remember Guids? They
are unique and can be considered random enough for most scenarios, that regular developer meets
in her life. But what if we need int?  Here is a solution: 

{% highlight csharp %}
[Fact]
public void more_random()
{
    var r1 = Guid.NewGuid().GetHashCode();
    var r2 = Guid.NewGuid().GetHashCode();
    r1.Should().NotBe(r2);

    // and even this:
    var rnd1 = new Random(r1).Next();
    var rnd2 = new Random(r2).Next();
    rnd1.Should().NotBe(rnd2);
}
{% endhighlight  %} 

Yes. `Guid.GetHashCode()` should be your friend if you need something _more random_ than default
`new Random()`.

> And if you need to generate random objects, for testing or any other purpose, consider using 
[AutoFixture](https://github.com/AutoFixture/AutoFixture). It can create random object graphs, 
collections and many more.
