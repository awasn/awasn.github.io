---
title: "<![CDATA[]]> kills VS2K5"
date: 2007-11-06T22:04:00+01:00
lastmod: 2018-06-07T16:08:00+02:00
languageCode: pl-pl
description: Nawiasy ostre w komentarzu do kodu C# zawieszają Visual Studio 2005
categories:
  - Development
  - Tools
tags:
  - .NET
  - Visual Studio
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/11/06/cdata.aspx
disqus_disable: true
---

Królestwo za nawiasy ostre w komentarzu, który zawiera powyższą sekcję. Nie ma znaczenie gdzie będziemy je próbowali wstawić. Czy w samej sekcji CDATA, czy też w innym miejscu w ramach istniejącego już komentarza. Visual Studio 2005 robi kaput.

PS. Podziękowania dla [Michała](http://zine.net.pl/blogs/mgrzeg/default.aspx), który brał udział w eksperymencie i robił za przysłowiową świnkę doświadczalną.

---

Z powyższego tekstu nie wynika to jasno, ale chodzi o komentarze do kodu C#:

```csharp
    /// <summary>
    /// <![CDATA[]]>
    /// </summary>
    /// <param name="args"></param>
    static void Main(string[] args)
    {
    }
```

Próba wstawienia nowej linii komentarza `/// <` doprowadzi do zawieszenia Visual Studio.