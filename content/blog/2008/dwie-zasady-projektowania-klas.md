---
title: Dwie zasady projektowania klas
date: 2008-11-06T23:22:00+01:00
lastmod: 2018-06-08T17:12:00+02:00
languageCode: pl-pl
description: Zwiększenie wydajności klas w .NET Compact Framework poprzez rezygnację z właściwości na rzecz pól
categories:
  - Development
tags:
  - .NET
  - .NET Compact Framework
  - Design patterns
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/11/06/dwie-zasady-projektowania-klas.aspx
disqus_disable: true
---

Jedna z zasad dobrego projektowania klas to rezygnacja z pól na rzecz właściwości. Jedna z zasad wydajnego programowania (dotyczy zwłaszcza .NET Compact Framework) przy tworzeniu klas to rezygnacja z właściwości na rzecz pól.

---

[.NET Compact Framework version 2.0 Performance and Working Set FAQ](https://blogs.msdn.microsoft.com/netcfteam/2005/05/04/net-compact-framework-version-2-0-performance-and-working-set-faq/): "Simple property access can be inlined by JIT, but no assumptions should be made about this [...] Accessing fields directly normally results in better performance"