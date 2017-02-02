---

layout: post
title: 'Teamcity: Extract Parameters from File'
date:   2017-02-02 14:25:00 +0200
categories: Teamcity Powershell Regex

---

During setting up continuous deployment (there ~~should~~
will be a dedicated post about that) we need to extract
the version of JS code being packed into [nuget](https://www.nuget.org/) 
package. We use [Octopus](https://octopus.com/) for our deployments --
that's where nuget comes from. 

Our FE guys store version info in `.env` file. This is a
text file having format very close to [`*.ini`](https://en.wikipedia.org/wiki/INI_file)
where version there is stored as `VERSION=1.0.0.1`. So
the task is to extract that version. It turned out that
Teamcity does not support that out of the box, or at least
this functionality is not obvious. The solution emerged
quickly -- we need to write some code (are we developers or 
not?!). All our agents are Windows machines, thus Powershell 
is our friend. 

**TL;DR;** Here the script:

{% highlight powershell %}
select-string -path .\.env -Pattern 'VERSION=(.*)$' -allmatches | 
  % {$_.Matches.Groups[1].Value} | 
  % {write-host "##teamcity[setParameter name='ui.version' value='$_']"}
{% endhighlight %}

And some explanation:

- `select-string` is clone of [grep](https://en.wikipedia.org/wiki/Grep). It searches
strings in variety of places, including files
    - `-path` tells what file the search should be performed in
    - `-Pattern` (casing in powershell does not matter -- it's Windows :) ) specifies...
pattern. It uses standard .NET Regex [dialect](https://msdn.microsoft.com/en-us/library/az24scfc(v=vs.110).aspx)
Our pattern basically matches text starting with _VERSION=_ then any text up to the end
of the line. And that any text is the value of group 1.
- `|` is a pipe. _Piping_ means sending the result of previous command (left side of
the pipe) to the next command (right side of the pipe)
- `% { ... }` is a shortcut for `ForEach-Object` that literally runs code inside
curly braces for every item. `$_` is a variable containing the value of that item.  
Result of `select-string` is an array of matched strings and it is processed here item by
item.
- `$_.Matches.Groups[1].Value` returns the content of group 1 or every matched string
- `write-host` is aka `Console.Out.WriteLine`. Here it produces a [service message](https://confluence.jetbrains.com/display/TCD10/Build+Script+Interaction+with+TeamCity) 
that [instructs] (https://confluence.jetbrains.com/display/TCD10/Build+Script+Interaction+with+TeamCity#BuildScriptInteractionwithTeamCity-changingBuildParameterAddingorChangingaBuildParameterfromaBuildStepAddingorChangingaBuildParameter)
Teamcity to create or update build parameter `ui.version` and set it's value to the found string.

So we added a [Powershell build step](https://confluence.jetbrains.com/display/TCD10/PowerShell)
with this script and down the pipeline we can refer to that paramater as `%ui.version%`.

Of course that trick can be used not only with your build server but in everyday
activity. Next time when you need to quickly find some text in your files -- give a
try to `select-string`! 