---
title: SQL Server 2005 Compact Edition
date: 2007-09-13T14:13:00+02:00
lastmod: 2018-09-17T14:23:00+02:00
languageCode: pl-pl
description: Konfiguracja ASP.NET do korzystania z SQL Server 2005 Compact Edition bez rzucania wyjątkiem NotSupportedException
categories:
  - Databases
tags:
  - .NET
  - ASP.NET
  - SQL Server
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/09/13/sql-server-2005-compact-edition.aspx
disqus_disable: true
---

Pewną nowością było wprowadzenie przez Microsoft na komputery typu PC, dostępnej do tej pory we wcześniejszych wersjach w ramach urządzeń mobilnych, bazy danych SQL Server 2005 Compact Edition (`http://www.microsoft.com/sql/editions/compact/default.mspx`), która nie wymaga instalacji żadnego serwera bazodanowego a do obsługi potrzebuje jednej biblioteki System.Data.SqlServerCe.dll. Dzięki temu, zamiast przechowywać dane niezbędne do pracy aplikacji np. w plikach XML możemy skorzystać z możliwości silnika SQL. Z tego typu sytuacjami niejednokrotnie się spotykamy. Instalacja dowolnego serwera nie jest możliwa, a jednocześnie szukamy możliwości sprawnego zarządzania niedużą ilością danych.

Osoby chcące wykorzystać SQL Server 2005 Compact Edition w ramach aplikacji ASP.NET spotka niestety przykra niespodzianka. Próba nawiązania połączenia z bazą danych spowoduje bowiem wygenerowanie wyjątku `NotSupportedException: SQL Server Compact Edition is not intended for ASP.NET development`. Okazuje się na szczęście, iż ograniczenie to można łatwo obejść wywołując w ramach kodu poniższą metodę:

```csharp
            AppDomain.CurrentDomain.SetData(
                "SQLServerCompactEditionUnderWebHosting",
                true);
```

Polecam także zapoznanie się z konkurencyjnym rozwiązaniem [Embedded Firebird](https://firebirdsql.org/pdfmanual/html/fbmetasecur-embedded.html), które również posiada biblioteki przeznaczone na platformę .NET oraz możliwość integracji ze środowiskiem Visual Studio. Wśród zalet silnika [Firebird](https://firebirdsql.org) jest między innymi brak ograniczeń co do rozmiaru bazy danych. Jako ciekawostkę podam fakt, iż tę wbudowaną bazę danych wykorzystuje między innymi AVG Anti-Virus.