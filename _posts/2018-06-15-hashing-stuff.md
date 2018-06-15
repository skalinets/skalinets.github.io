---
title: Quick and Dirty Hashing in ASP.NET Core
---

One of our product imports data from other system via SFTP (you might confuse it [with FTPS](https://www.secureblackbox.com/kb/articles/FTPS-vs-SFTP.rst), 
but that is OK). Data coming from that system contains the whole dataset every time 
rather than just updates since last upload and does not have timestamps. So, we
need to identify rows that should be updated.

Obvious solution would be using of hash. During import we calculate the hash of each
row, compare it with saved one and update our DB only if hashes are different. The
question is how to compute a hash in ASP.NET?

In good old days I used to use something like

{% highlight csharp %}
var value = "my;csv;string";

var algo = HashAlgorithm.Create("SHA512");
var hb = algo.ComputeHash(Encoding.UTF8.GetBytes(value));
var hash = Convert.ToBase64String(hb);
{% endhighlight%}

However this time it raised an exception: 

    System.PlatformNotSupportedException: Operation is not supported on this platform.
      at System.Security.Cryptography.HashAlgorithm.Create(String hashName)

Bummer. I tried to google it, found some huge discussion in [dotnet repo](https://github.com/dotnet/corefx/issues/22626), started to read it and... **completely missed the workaround from the very first post** :) But here is
where the fun begins.

Further readings brought me to [that](https://www.c-sharpcorner.com/article/hashing-in-asp-net-core-2-0/):

{% highlight csharp %}
var hashBytes = KeyDerivation.Pbkdf2(
    value,
    salt: Encoding.UTF8.GetBytes("salt"),
    prf: KeyDerivationPrf.HMACSHA512,
    iterationCount: 1000,
    numBytesRequested: 256 / 8);

var hash = Convert.ToBase64String(hashBytes);
{% endhighlight%}

Yes, I intentionally added argument names, otherwise argument values would look
meaningless (they are anyway). It is supposed to be cryptographically secure and works, but seems to be too much for just computing a hash. 

Blog post revealing this beast claims that it is used in ASP.NET Identity and
I remember that hashing password is much easier. So the question: can we use
famous [PasswordHasher<>](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.identity.passwordhasher-1?view=aspnetcore-2.1) for our simple purpose?

Here comes the code:

{% highlight csharp %}
var hashPassword = new PasswordHasher<object>()
                        .HashPassword(new object(), value);
{% endhighlight%}

Well, it smells like a hack. But it is still oneliner, easy to use and working.

Also you can use hash verification:

{% highlight csharp %}
var r = new PasswordHasher<object>()
          .VerifyHashedPassword(new object(), hashPassword, value);
var isTheSame = r == PasswordVerificationResult.Success;
{% endhighlight%}

However it is less attractive because of messing with `PasswordVerificationResult`. I'd rather compute hash again and compare it with
a stored one.

What do you think? :)

**PS** Those who don't like that approach still can use the very first snippet in 
this post, but change first line. You still would need to convert to and from Base64 and compare

{% highlight csharp %}
var algo = (HashAlgorithm) CryptoConfig.CreateFromName("SHA512");
var hb = algo.ComputeHash(Encoding.UTF8.GetBytes(value));
var h = Convert.ToBase64String(hb);
{% endhighlight%}
