---
title: Debugger.Break ładuje pakiety
date: 2008-06-04T13:34:00+02:00
languageCode: pl-pl
description: Wymuszenie debugowania kodu .NET C# w Visual Studio
categories:
  - Development
tags:
  - .NET Compact Framework
  - Mobile
  - Software testing
  - Visual Studio
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/06/04/debugger-break-aduje-pakiety.aspx
disqus_disable: true
---

W czasie testowania oprogramowania mobilnego zdarza się, iż zdalne debuggowie przestaje funkcjonować. Obok ustawionych punktów wstrzymania pojawia się zaś ikona, która po najechaniu kursorem wyświetla komunikat, iż zdalna praca jest niemożliwa, ponieważ dany pakiet nie został załadowany. Co ciekawe dalej może rozpocząć wykonywanie krokowe aplikacji. Restart Visual Studio w takiej sytuacji nie pomaga.

Okazuje się, iż rozwiązanie problemu jest trywialne. Należy w kodzie programu, gdzie chcemy ustawić punkt wstrzymania wstawić następujące polecenie:

```csharp
            Debugger.Break();
```

Dzięki temu debugger zostanie przywrócony do żywych.