---
title: Day of investigation, one line to fix
---

This kind of stories are happening everywhere in developer community. You face the bug,
spend hours or even days to nail it, and result is some stupid small fix.

My case is interesting because my bug was the `NullReferenceException` in 3rd party code.
It is even more exciting because bug was reproduced only when code was running in Azure Web
Application. Locally it was OK.

The bug itself and fix is not that unique. You can find more details in [github issue][1].
That said you might find interesting some approaches that we used during troubleshooting.

In one of our projects we need video calls. Our decision was to use some service for that and
we decided to try [tokbox](http://tokbox.com). They have nice onboarding experience, good API and the flow is
extremely easy to get start with. So we created an account, added a code from samples to our project and
got working video chat in our local dev boxes. However, when deployed to Azure, it was not working
because of `NullReferenceException` inside tokbox's client code.

Since it was working fine at local boxes, the first thing to investigate was the misconfiguration.
We double checked it and it turned out to be correct.

Then we asked ourselves "could it be something in our code that breaks things"? We created a brand new
project in Visual Studio, added video chat code there, deployed it and... it was working fine. It
gave us a hope that it is possible in general and we just need to push harder.

How to get inside 3rd party code running in another data center? If you think I now will start
talking about remote debugging -- no, I will not. I don't like debugging at all, and absolutely sure
that it has it's limits. You may spend a lot of time trying to attach debugger to remote server, but
hardly you would be able to debug on prod. Apparently it is more efficient to use technics that can
work everywhere.

One of such technics is tracing. Fortunately, tokbox' .NET SDK has debug mode. Unfortunately,
it writes all debug [information to `stout` using `Console.WriteLine`](https://github.com/opentok/Opentok-.NET-SDK/blob/master/OpenTok/Util/HttpClient.cs#L255). 
That does not help too much in ASP.NET application.

But we can redirect the output of `Console.WriteLine` to some `TextWriter`, or, more precisely,
`StringWriter` and out it's content to logs.

Another issue that emerged was that we needed to deploy code changes to our staging each time we 
made them. The feedback cycle length was not that great, since we have a CI/CD process with code review,
PR build checks and staged deployments (first dev, then staging). It works fine, but usually is taking
~15 minutes to complete. When you make change and keen to see it in action, waiting 15 minutes is not
great.

So we decided to switch to manual deployment. Yes -- right click / publish just from Visual Studio.
However our deployment routine was more complicated -- we have a bunch of different services,
databases, storage accounts that are being set up during our automated deployment. After Visual
Studio publish our app is not properly configured and we cannot even login to it to reproduce the issue.

But wait, do we need the whole app to troubleshoot this single issue? Apparently not. We just added a
TestController with the following code:

{% highlight csharp%}
[AllowAnonymous]
public class TestController : ApiController
{
    [Route("test")]
    public string Get()
    {
        var stringWriter = new StringWriter();
        Console.SetOut(stringWriter);
        var openTok = new OpenTok(key, secret) {Debug = true};
        try
        {
            openTok.CreateSession();
        }
        catch { }
        stringWriter.Flush();
        return stringWriter.ToString();
    }
}
{% endhighlight %}

This nice small anonymous controller allowed us to get the actual error and create [this issue][1] that
helped to troubleshoot and add that single line of code fixing our pain:


{% highlight csharp%}
System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;
{% endhighlight %}

And yes, this problem is well-known. I used to apply this piece of code in other projects few times already.
And there is a [KB article][2] dedicated to it that we should have found at the very beginning.

But you never know if you faced known issue or not. Being prepared and having a bunch of potential next steps
to troubleshoot it is a very important (I would even say critical) skill of software developer. 


[1]: https://github.com/opentok/Opentok-.NET-SDK/issues/108
[2]: https://support.tokbox.com/hc/en-us/articles/360000373020-Getting-error-while-creating-session-using-OpenTok-Net-SDK
