---
layout: post
title:  "StringBuilder is just a Cargo Cult (sometimes)"
date:   2018-01-10 17:55:31 +0200
comments: true

---

Can you think of top 10 questions in .NET developer interview? I bet one of them is about string builders. 
Like "how would you concatenate multiple strings?". With the correct answer "I'd use `StringBuilder`. Because,
you know, `string` is immutable, allocations, blah, blah, blah...". And if you pretend to be an experienced
developer, your code has 0 (zero) concatenations via `+`. 

Even when it comes to join 2 (3, 4, 5, ...) literals or constants, they still use `StringBuilder`. Or course, it's
less effective, because such concatenations usually happen during compilation (constants are embedded into code as
literals).

But what about regular strings? Should one always use `StringBuilder`? I've prepared a small benchmark, using
[BenchMarkDotNet](http://benchmarkdotnet.org/) (source code
is available on [GitHub](https://github.com/skalinets/StringBuilderTest)) that shows that when the number of strings
is less than 20, `StringBuilder` has no advantage over `+`. Even more, when number of items is less that 10, `+` has
better performance. And this is exactly the most used scenario of string concatenation. If you have more than 10 items,
you'd most likely use the loop with `StringBuilder`, or `String.Join()` or (if you're crazy badass) `IEnumerable.Aggregate()`. 

Here are results from tests running on my machine:

``` ini

BenchmarkDotNet=v0.10.11, OS=Windows 10 Redstone 3 [1709, Fall Creators Update] (10.0.16299.192)
Processor=Intel Core i7-4720HQ CPU 2.60GHz (Haswell), ProcessorCount=8
Frequency=2533208 Hz, Resolution=394.7564 ns, Timer=TSC
.NET Core SDK=2.1.2
  [Host]     : .NET Core 2.0.3 (Framework 4.6.25815.02), 64bit RyuJIT
  DefaultJob : .NET Core 2.0.3 (Framework 4.6.25815.02), 64bit RyuJIT


```

|                    Method | NumberOfItems |        Mean |        Error |       StdDev |      Median |
|-------------------------- |-------------- |------------:|-------------:|-------------:|------------:|
|       **StringBuilderAction** |             **2** |    **885.5 ns** |     **7.824 ns** |     **7.319 ns** |    **885.3 ns** |
| StringConcatenationAction |             2 |    770.6 ns |     6.802 ns |     6.362 ns |    771.4 ns |
|       **StringBuilderAction** |             **5** |  **1,785.5 ns** |    **10.304 ns** |     **9.639 ns** |  **1,784.6 ns** |
| StringConcatenationAction |             5 |  1,674.5 ns |    20.619 ns |    18.278 ns |  1,672.6 ns |
|       **StringBuilderAction** |            **10** |  **3,215.0 ns** |    **63.250 ns** |    **62.120 ns** |  **3,186.8 ns** |
| StringConcatenationAction |            10 |  3,313.1 ns |   102.415 ns |    85.521 ns |  3,283.2 ns |
|       **StringBuilderAction** |            **20** |  **5,951.5 ns** |    **48.102 ns** |    **40.167 ns** |  **5,937.8 ns** |
| StringConcatenationAction |            20 |  6,947.7 ns |    34.813 ns |    29.071 ns |  6,940.4 ns |
|       **StringBuilderAction** |            **50** | **14,997.1 ns** |   **347.515 ns** |   **985.841 ns** | **14,672.7 ns** |
| StringConcatenationAction |            50 | 22,794.2 ns |   474.567 ns | 1,291.094 ns | 22,303.9 ns |
|       **StringBuilderAction** |           **100** | **27,873.3 ns** |   **481.219 ns** |   **375.704 ns** | **27,864.9 ns** |
| StringConcatenationAction |           100 | 59,897.8 ns | 1,948.882 ns | 3,091.133 ns | 59,045.8 ns |



There is another, poor man's benchmark available on [dotnetfiddle.net](https://dotnetfiddle.net/rDf7hm). It might be less accurate,
but `+` always wins.

Having that we can conclude that using `StringBuilder` to just concatenate a couple of strings is nothing but a [Cargo Cult](https://en.wikipedia.org/wiki/Cargo_cult).
More code and less performance...

Happy coding!