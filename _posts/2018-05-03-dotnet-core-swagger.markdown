---
title: 'Swagger Tips for ASP.NET Core'
---


When you do HTTP API (you can call them REST or even RESTful, but you'd [probably be wrong](https://www.google.com.ua/search?q=your+api+is+not+restful)), it is nice to have a documentation. While Microsoft
has [some home-brewed solution](https://docs.microsoft.com/en-us/aspnet/web-api/overview/getting-started-with-aspnet-web-api/creating-api-help-pages), and many .NET guys are OK with it, it is too far from being good.

Much better is [Swagger](https://swagger.io/). As it's site states, _Swagger is the world’s largest framework of 
API developer tools for the OpenAPI Specification(OAS), enabling development across the entire API lifecycle, from design and documentation, to test and deployment._
And that is true. It goes far beyond just documentation of your API, but still is very handy in that. It is well
known and widely adopted. If your API has Swagger, almost any client can connect to it with minimal efforts.

To use Swagger in .NET you need a [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore). Just 
follow their readme and you'll get your API covered in minutes. I won't repeat it here, since documentation is rather good.

Instead I will highlight few areas that might not be obvious from the docs.

Return Types
----
ASP.NET Core is very flexible. In some cases it seems to be too flexible. For instance, your
controller methods could return either `IActionResult` or typed result. Typed
results might seem to be a better choice, but apparently they are not, if you 
care about beyond-happy-path scenarios. You might want to do `return BadRequest(...)` or `return NotFound()` that typed methods wouldn't welcome. However, `IActionResult` can do anything. 

But that adds burden on you as developer to specify return type explicitly, so that
Swagger could pick it up and build nice spec and UI around it. To do that use `ProducesResponseType`: 

{% highlight csharp %}
[ProducesResponseType(typeof(IDictionary<string, string[]>), 400)]
[ProducesResponseType(typeof(LoginResponse), 200)]
public async Task<IActionResult> Register([FromBody] RegistrationModel model)
{% endhighlight %}

In above example we are telling Swagger that our controller method can return either 200 OK with `LoginResponse`, or 400 Bad Request with some dictionary. 

BTW that dictionary is a result of `return BadRequest(ModelState)` and we can use built-in
`ModelState` to nicely return different errors:

{% highlight csharp %}
...
ModelState.AddModelError(nameof(email), "Email is already registered");
return BadRequest(ModelState);
...
{% endhighlight %}

That dictionary is even a special class in ASP.NET Core, but I forgot it's name and it is just a wrapper around `IDictionary<string, string[]>`.

Authorization
---

One of cool features of Swagger is that it produces nice UI, where you can inspect
methods, types and even try them out. It works really nice until you want to
try endpoint that requires authorization. Up until the latest update, Swagger UI allowed
users to paste JWT token, that could be used for testing your API, however with the latest update that UI has gone.

It turned out that UI was displayed out of the box [because of bug](https://github.com/domaindrivendev/Swashbuckle.AspNetCore/issues/603#issuecomment-368087943). And to be able to use it you 
need to add some more code to your configuration. Fortunately, the amount of 
code is rather small. First you copy&paste the code from the comment above, and then just add another few lines to Swagger configuration:

{% highlight csharp %}
services.AddSwaggerGen(_ =>
{
    _.DocumentFilter<SecurityRequirementsDocumentFilter>();
    _.AddSecurityDefinition("Bearer", new ApiKeyScheme
    {
        Name = "Authorization",
        In = "header"
    });
...
{% endhighlight %}

The result is a new **Authorize** button, that once clicked, allows to enter your JWT token. Note that you need to explicitly add *Bearer* before your token just like in Authorization header.

Enums
---

By default enums will appear in your Swagger documentation as numbers. To use strings instead, add call to `_.DescribeAllEnumsAsStrings();` in your invocation of `AddSwaggerGen()`.

Summary
---

OK, that seems to be all for today. If you have other useful settings for Swagger, just let me know.

Happy coding!