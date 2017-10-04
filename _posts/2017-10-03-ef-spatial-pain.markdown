---

layout: post
title: "Spatial Pain with Entity Framework"
date: 2017-10-05 12:00 +02:00
categories: EF .NET

---

In one of our project there is a requirement to sort addresses by the distance to a given point. We use Azure SQL Database
as a storage and Entity Framework 6.1.3 as an ORM. 

In 2017 all modern databases [seem](https://www.scribd.com/presentation/2569355/Geo-Distance-Search-with-MySQL) to support 
spatial calculations, so does Azure SQL Database. Moreover, Entity Framework also allows to use geo operations in its
queries. After all it's just a bunch of [sines and cosines](https://www.scribd.com/presentation/2569355/Geo-Distance-Search-with-MySQL) -- nothing complicated, right? 

Well, not necessarily.

First I tried [Microsoft's tutorial](https://msdn.microsoft.com/en-us/data/hh859721) and immediately got a [crazy error](https://stackoverflow.com/questions/19551142/entitytype-dbgeography-has-no-key-defined). "Has no key defined", WAT? OK, nailed it, thanking to 
Stackoverflow. The reason was a typo in tutorial (it is still there).

Next attempt -- a new error. 
{%highlight json%}
error 2: {"Spatial types and functions are not available for this provider because the assembly 'Microsoft.SqlServer.Types' version 10 or higher could not be found. "}
{%endhighlight%}

Sooo, I need to install some stuff to perform spatial calculations. That are sines and cosines as we remember. That's Microsoft's way,
what can I say :)

I tried to install it via [chocolatey](https://chocolatey.org/packages/sql2016-clrtypes). But it still didn't work. Maybe reboot (typical next step
when something is not properly working after installation on Windows)? No luck..

[Here](https://stackoverflow.com/questions/43221467/assembly-microsoft-sqlserver-types-version-10-or-higher-could-not-be-found) some kind guy posted link to
CLR types for SQL Server 2012 (!). After I had installed those it started to work in console application, but apparently not in ASP.NET (where is a headbang emoji?).

It turned out that IIS should work in 32 bit mode. I found some SO comment (link is lost) saying that some x86 magic should be done with IIS. In my case it was IIS Express and I didn't know how to enable 32bit applications there. Instead, I just installed x86 version of SQL CLR types and it started to work.

{%highlight csharp%}
.OrderBy(l => l.Address.Geocode.Distance(booking.Address.Geocode))
{%endhighlight%}

So, the conclusion: You can use spatial queries with MS SQL and Entity Framework, but... It's not that trivial.