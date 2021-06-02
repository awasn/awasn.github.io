---
title: Refaktoryzacja metod zwrotnych
date: 2008-07-20T00:57:00+02:00
languageCode: pl-pl
description: Upraszczamy w .NET C# kod przez usunięcie wszystkich własnych definicji delegatów będących metodami zwrotnymi i zastąpienie ich przez Action oraz Func
categories:
  - Development
tags:
  - .NET
  - Design patterns
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/07/20/refaktoryzacja-metod-zwrotnych.aspx
disqus_disable: true
---

Najnowsza refaktoryzacja kodu jednego z moich projektów polegała na usunięciu wszystkich własnych definicji delegatów będących metodami zwrotnymi. Zamiast tego użyłem standardowych metod z przestrzeni nazw `System`:

* `Action`;
* `Action<T>`;
* `Action<T1, T2>`;
* `Action<T1, T2, T3>`;
* `Action<T1, T2, T3, T4>`.

dla metod zwrotnych, które nie zwracają wartości oraz:

* `Func<TResult>`;
* `Func<T, TResult>`;
* `Func<T1, T2, TResult>`;
* `Func<T1, T2, T3, TResult>`;
* `Func<T1, T2, T3, T4, TResult>`.

dla metod zwrotnych zwracających wartość.

Dzięki temu zniknęło kilka klas. A sam kod wbrew pozorom stał się bardziej przejrzysty przez to, iż nie trzeba zastanawiać się jak wygląda metoda `WorkflowTerminateCallback` jeśli teraz w kodzie występuje wywołanie metody `Action`. Wbrew pozorom, ponieważ nazwa metody `WorkflowTerminateCallback` jest bardziej wymowna od metody `Action` i niepokoiło mnie, czy przypadkiem źródła nie staną się mniej przejrzyste. Kod żyje jednak w pewnym kontekście i po wprowadzeniu zmian okazało się, iż moje obawy były bezzasadne.