cargo cult: always use string builder

{% highlight csharp %}
using System;
using System.Linq;
using System.Diagnostics;
using System.Text;
					
public class Program
{
	public static void Main()
	{
		var ss = Enumerable
					.Range(1, 5000)
					.Select(_ => new { 
									s1 = Guid.NewGuid().ToString(), 
									s2 = Guid.NewGuid().ToString(),
									s3 = Guid.NewGuid().ToString(),
									s4 = Guid.NewGuid().ToString(),
									s5 = Guid.NewGuid().ToString(),
									s6 = Guid.NewGuid().ToString(),
					})
					.ToList();
		
		var sw = Stopwatch.StartNew();
		ss.ForEach(_ => Add2(_.s1, _.s2, _.s3, _.s4, _.s5, _.s6));
		Console.WriteLine("String Builder: {0}", sw.Elapsed);
	
		sw = Stopwatch.StartNew();
		ss.ForEach(_ => Add1(_.s1, _.s2, _.s3, _.s4, _.s5, _.s6));
		Console.WriteLine("String  Concat: {0}", sw.Elapsed);
	}
	
	static string Add1(string s1, string s2, string s3, string s4, string s5, string s6) { return s1 + s2 + s3 + s4 + s5 + s6; }
	
	static string Add2(string s1, string s2, string s3, string s4, string s5, string s6)
	{
		var sb = new StringBuilder();
		sb.Append(s1);
		sb.Append(s2);
		sb.Append(s3);		
		sb.Append(s4);		
		sb.Append(s5);		
		sb.Append(s6);
		return sb.ToString();
	
	}
}
{% endhighlight%}

https://dotnetfiddle.net/rDf7hm

