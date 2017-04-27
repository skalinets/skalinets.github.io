---

layout:  post
title: Approval Tests 


---

Unit tests are great tool for development. Combined with such practice as TDD they can significantly
improve your development experience. 



{% highlight csharp %}
public class AutoApproveReporter : IApprovalFailureReporter
{
    public void Report(string approved, string received)
    {
        File.Copy(received, approved, true);
    }
}
{% endhighlight %}