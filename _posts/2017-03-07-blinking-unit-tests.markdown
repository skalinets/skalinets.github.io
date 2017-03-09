---

layout: post
title: "How to deal with blinking tests"
date: 2017-03-07 17:52:31 +0200
category: .NET C# "Unit Tests"

---

I think every developer who uses unit tests is aware of "blinking" or unstable tests. Those
are tests with non-deterministic behavior. They can either pass or fail from run to run even
without code changes. Such test are annoying and troubleshooting them could be a nightmare. 

Let's consider a naive example:
{% highlight csharp %}
public class BlinkingInside
{
    int randomValue = new Random().Next(100);

    [Fact]
    public void i_am_blinking()
    {
        randomValue.Should().BeLessOrEqualTo(70);
    }
}
{% endhighlight  %} 

Here I am using [xUnit2](https://xunit.github.io/) and [Fluent Assertions](https://github.com/fluentassertions/fluentassertions).
This test is stupid but is good enough for demo purposes: it (very rarely) blinks. How to troubleshoot it?

The first step of troubleshooting is reproducing. We basically need to observe falling test,
check it's variables and figure out what's is going on. The problem is that brute force 
reproducing here might not work. We would just spent a lot of time running test again and
again just to spot one fail of 100s (or even more). 

OK, but we're developers -- guys who can automate boring stuff! If there just was the way of
running the same piece of code many times... Oh, wait, loops -- these creatures of imperative
programming should help us. So, we're adding some loop:

{% highlight csharp %}
...
    public void i_am_blinking()
    {
        for (int i = 0; i++; i < 10000)
            randomValue.Should().BeLessOrEqualTo(70);
    }
...
{% endhighlight  %} 

And.. nothing happens. You see, the problem is in `randomValue` that is initialized outside of the 
loop and method. Our code just checks the same values again and again. We need to be a little more 
smart -- and run test method itself several times. xUnit re-instantiates the test class before each 
test run, so if we run the test method multiple times, it will receive new `randomValue` for each run.

With previous version of `xUnit` it was pretty simple to achieve that goal. You just needed to create
your custom attribute and copy-paste [few lines](http://www.bricelam.net/2012/04/xunitnet-extensibility.html#factattribute) there.
`xUnit2` was reworked and that extensibility point has been changed. Now you need more work to do.

But there is one _dirty_ hack. Instead of making custom attributes we can use `Theory` with `MemberData`
that generates needed number of runs. This is simple as:

{% highlight csharp %}
public class BlinkingInside
{
    public static readonly IEnumerable<object[]> Foo = Enumerable.Range(1, 1000).Select(i => new object[] { i });

    private static Random rnd = new Random();
    private readonly int randomValue = rnd.Next(100);

    [Theory]
    [MemberData(nameof(Foo))]
    public void i_am_blinking(int _)
    {
        randomValue.Should().BeLessOrEqualTo(70);
    }
}
{% endhighlight  %} 

Note that property or field used in `MemberData` should be `public static` and return `IEnumerable<object[]>`. 
Another thing worth noting is that we added one argument to test method signature. Otherwise, all tests
will fail. 

> The first version of code snippet above contained a bug that is discussed in this [post](know-your-random).

Once we reproduced failing test we can add any logging / debugging etc, fix the issue, check it with our
multirunner and revert it back to `[Fact]`.
