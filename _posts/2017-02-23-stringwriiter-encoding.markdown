---
layout: post
title:  "Readonly StringWriter.Encoding: should I care?"
date:   2017-02-23 14:40:31 +0200
categories: .NET csharp BCL
---

During code review we spotted a class inherited from `StringWriter`. The only intention of that
was the need to override `StringWriter.Encoding` property that was read-only for some reason.
The use case was the XML serialization. 

The code was like that (of course it was googled :) )

{% highlight csharp %}
private static Stream Serialize<T>(T serializableObject)
{
    using (var encodedWriter = new StringWriterWithEncoding(Encoding.UTF8))
    {
        var xmlSerializerNs = new XmlSerializerNamespaces();
        xmlSerializerNs.Add(string.Empty, string.Empty);
        var serializer = new XmlSerializer(serializableObject.GetType());

        serializer.Serialize(new XmlTextWriter(encodedWriter), serializableObject, xmlSerializerNs);
        return new MemoryStream(Encoding.UTF8.GetBytes(encodedWriter.ToString()));
    }
}

private class StringWriterWithEncoding : StringWriter
{
    public StringWriterWithEncoding(Encoding encoding)
    {
        Encoding = encoding;
    }

    public override Encoding Encoding { get; }
}

{% endhighlight %}  

The idea to inherit from BCL class always looks weird to me. Especially in this case. The first question:
why default `StringWriter.Encoding` doesn't work? And next one: why is it read-only?

It turned out that `Encoding` affects the `encoding` value of XML declaration. This code produces following:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
{% endhighlight %}

While standard `StringWriter` generates

{% highlight xml %}
<?xml version="1.0" encoding="utf-16"?>
{% endhighlight %}

OK, but why is it read-only? To figure it out I opened [source code](https://referencesource.microsoft.com/#mscorlib/system/io/stringwriter.cs)
of `StringWriter`. And figured out the following:

- It [wraps](https://referencesource.microsoft.com/#mscorlib/system/io/stringwriter.cs,79) a static private 
[field](https://referencesource.microsoft.com/#mscorlib/system/io/stringwriter.cs,36) that is essentially the 
`Encoding.Unicode`, that in turn is an [UTF-16 encoding](https://msdn.microsoft.com/en-us/library/system.text.encoding.unicode(v=vs.110).aspx)
- It is not used in the class itself

Seems like it is a completely useless thing. And `StringWriter` itself is just a wrapper around
`StringBuilder` that have no concept of encoding at all. As we might remember, internally .NET strings
[use UTF-16](http://csharpindepth.com/Articles/General/strings.aspx). And that Encoding is exactly what
is returned by `StringWriter.Encoding`. That is why it cannot (and should not) be changed -- because we 
cannot change internal encoding of `string`.

The fact that `XmlTextWriter` uses that property to render encoding in declaration seem to be OK, but this
particular usage is not. `StringWriterWithEncoding` just fools it's clients advertising different encoding. 
Total hack. 

But how to do it properly? In fact we don't need `StringWriter` at all. Curious reader can notice that the code
from the top just uses `StringWriter` to get a string and immediately after that takes bytes from that string to
`MemoryStream`. Why not use stream instead?

{% highlight csharp %}
private Stream Serialize<T>(T serializableObject)
{
    var stream = new MemoryStream();
    var encodedWriter = new StreamWriter(stream, Encoding.UTF8);

    var serializer = new XmlSerializer(serializableObject.GetType());
    serializer.Serialize(new XmlTextWriter(encodedWriter), serializableObject);
    stream.Seek(0, SeekOrigin.Begin);

    return stream;
}
{% endhighlight %}
