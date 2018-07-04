---
title: How to Build a Link to SPA Site in ASP.NET Core
---

Problem
--

When your web application sends emails to its users they might need to contain
links to the site. Like "click here to get more details" etc. If you use
MVC it's not a problem, since you can use a method of [HttpContext.Request](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.extensions.urihelper.getencodedurl?view=aspnetcore-2.1#Microsoft_AspNetCore_Http_Extensions_UriHelper_GetEncodedUrl_Microsoft_AspNetCore_Http_HttpRequest_) 
to get the address of request and then change it as needed.

It might work also with SPA, however in some cases it doesn't. For example, during development your client site can be running on another port
than your server. Because you probably use `ng serve` for Angular or any other
tool relevant to your FE stack. And server can be either running locally via
`dotnet run`, or you could connect to remote server. However you'd like to
get links in your emails leading to your client site, not remote one.

Sub-optimal Solution
---

To workaround this issue we used something like:

{% highlight csharp %}
IHttpContextAccessor accessor;
...
var requestUri = new Uri(accessor.HttpContext.Request.GetEncodedUrl();
var apiRoot = GetApiRoot(requestUri);
return apiRoot + $"/here-is-you-path?with=params";
...

private string GetApiRoot(Uri requestUri) => env.IsDevelopment() ? "http://localhost:4200" : $"{requestUri.Scheme}://{requestUri.Authority}";

{% endhighlight %}

It was neither elegant nor robust. And it did not work for remote servers, since it is not running in development mode.

Better Solution
---

At some point we realized that we could use another request property. It is [Referer header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer). 

> Funny fact is that `Referer` is a misspelled **Referrer**, and not knowing
that fact might cost few hours or debugging, when one can, for example, try
to find a header by orthographically correct string `Referrer`. .NET Framework
adds more confusion having [HttpRequest.UrlReferrer](https://msdn.microsoft.com/en-us/library/system.web.httprequest.urlreferrer%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396)

As result of our refactoring we've got nice method:

{% highlight csharp %}
public string GetClientLink(string path, string query)
{
    var headers = accessor.HttpContext.Request.Headers;
    var refererAddress = headers[nameof(HttpRequestHeader.Referer)].ToString();
    return new UriBuilder(refererAddress)
    {
        Path = path,
        Query = query
    }.ToString();
}
{% endhighlight %}

Note the using of combination of constructor arguments and object initializers
in a single statement. In this case it allows us to get the full referrer url
and replace path and query with our values. It is very handy.

That's all for now. Please let me know if this was useful, or there are better ways to build a link :) 

Cheers!
