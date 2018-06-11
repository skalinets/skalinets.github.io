---
title: Swagger Authorization per Endpoint in ASP.NET Core

---

In my [recent post about Swagger][1] there was a recipe of how to add authorization. It works nice, however the result is that
all your endpoints are shown as protected (have lock icons) in the UI. First, it
did not seem to be an issue, since any of that lock icon appeared to do the same thing -- 
adding a auth token to ALL subsequent requests. No matter what icon you click -- from the
header or from any operation, result would be the same.

But when we tried to generate a client code from our swagger spec, it turned out that
generated code adds authorization to all endpoints. Including ones supposed to be anonymous
(`/login`, `/register` etc.) And that provoked further investigation of Swagger and Swashbuckle.

Apparently the code from [this post][1] is adding a security
requirement for the whole swagger document. But we need to add it only to protected endpoints.

We use [`[Authorize]` attribute][2]
to protect our endpoints. First, we were putting that attribute to all controllers / methods
that should not be available to anonymous users, but then we realized that almost all of them
should be protected. So maybe we should make them protected by default? It is rather easy in 
ASP.NET Core:

{% highlight csharp %}
// in Startup.cs ConfigureServices method
...
services.AddMvc(_ =>
    {
        var authPolicy = new AuthorizationPolicyBuilder()
            .RequireAuthenticatedUser()
            .Build();
        _.Filters.Add(new AuthorizeFilter(authPolicy));
    });
{% endhighlight %}

The above code adds [`[Authorize]` attribute][2]
to all endpoints. And we still can declare anonymous ones my decorating it with [[`AllowAnonymous`]][3]
It's all nice, but Swashbuckle does not recognize our attributes at all. It needs explicit instructions
regarding what authorization schemes are available to what operations. Fortunately, like in ASP.NET 
Core authorization, instead of marking each and every operation we can create a filter that applies
to all operations. Here the code:

{% highlight csharp %}
public class SecurityRequirementsOperationFilter : IOperationFilter
{
    public void Apply(Operation operation, OperationFilterContext context)
    {
        if (!context
              .ControllerActionDescriptor
              .GetControllerAndActionAttributes(true)
              .Any(_ => _ is AllowAnonymousAttribute))
        {
            operation.Security = new List<IDictionary<string, IEnumerable<string>>>
            {
                new Dictionary<string, IEnumerable<string>>
                {
                    {"Bearer", Array.Empty<string>()}
                }
            };
        }
    }
}

// in Startup.cs ConfigureServices method
...
services.AddSwaggerGen(_ =>
{
    _.OperationFilter<SecurityRequirementsOperationFilter>();
...
}

{% endhighlight %}

`SecurityRequirementsOperationFilter` is a small class that inspects the controller action it is being called for
and checks if it has [`[AllowAnonymous]`][3] attribute applied. If not, it adds a security requirement to that page. 

> If you don't use `AuthorizationFilter` and decorate protected endpoints with [`[Authorize]`][2], you might need to change
the `if` condition to make it work for you.

Once that change was done, our Swagger generator began to recognize protected / anonymous endpoints and stopped to 
put `Authorization` header to each one.  

[1]: {% post_url 2018-05-03-dotnet-core-swagger%}
[2]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute?view=aspnetcore-2.1
[3]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute?view=aspnetcore-2.1
