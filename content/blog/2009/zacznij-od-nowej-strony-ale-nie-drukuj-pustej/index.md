---
title: Zacznij od nowej strony, ale nie drukuj pustej
date: 2009-05-27T11:43:00+02:00
languageCode: pl-pl
description: Tips and tricks. Rozwiązanie problemu drukowania w Reporting Services niepotrzebnej pustej strony
categories:
  - Databases
tags:
  - Reporting Services
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/05/27/zacznij-od-nowej-strony-ale-nie-drukuj-pustej.aspx
disqus_disable: true
---

Jakiś czas temu podałem sposób na [rozwiązanie problemu]({{< ref "zacznij-od-nowej-strony" >}}) drukowania w ramach kontrolki **List** podraportów zaczynając każdorazowo od nowej strony. Efektem było niestety drukowanie na koniec pustej strony. Wydawałoby się, iż wystarczy jedynie kontrolkę **Rectangle** na koniec wyłączyć i marnotrawstwo papieru oraz nadszarpywanie naszej reputacji zostanie zlikwidowane. W tym celu właściwości **Hidden** przypisałem wyrażenie:

```vbnet
    =IIF(RowNumber(“DataSet”) < CountRows(“DataSet”), False, True)
```

Analiza jest trywialna: dopóki bieżący numer wiersza jest mniejszy od liczby wszystkich wierszy w danym zbiorze danych kontrolka **Rectangle** jest wyświetlana.

{{< figure src="images/Rectangle.png" title="Właściwości kontrolki Rectangle" >}}

Okazuje się jednak, iż jakiekolwiek wyrażenie powoduje ignorowanie wstawiania znaku końca strony (**PageBreakAtEnd = True**)! Prosty eksperyment polega na zamianie wartości **False** na wyrażenie **=False**. Ot błąd w implementacji (nota bene zdaje się, iż jest on od wersji Microsoft SQL Server 2000).

Rozwiązanie?!

Ech... Rozbicie tego na dwie kontrolki **Rectangle**. Pierwsza sprawdza i ustawia tylko właściwość **Hidden** oraz zawiera drugą kontrolkę **Rectangle**. Ta druga, wewnętrzna kontrolka dopiero ustawia **PageBreakAtEnd** na **True**.

PS. Opisywany problem dotyczy na pewno Reporting Services w wersji 2005. Pozostałych wersji nie sprawdzałem.