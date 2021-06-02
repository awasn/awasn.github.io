---
title: Nie wyrzucamy wyjątków poza Domain Model
date: 2008-03-25T11:47:00+01:00
lastmod: 2018-09-17T12:22:00+02:00
languageCode: pl-pl
description: Postulat aby nie wyrzucać wyjątków poza Domain Model
categories:
  - Development
tags:
  - .NET
  - Design patterns
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/03/25/nie-wyrzucamy-wyj-tk-w-poza-domain-model.aspx
disqus_disable: true
---

[Udi Dahan](http://udidahan.weblogs.us) na swoim blogu umieścił ciekawy [wpis](http://udidahan.weblogs.us/2008/02/29/how-to-create-fully-encapsulated-domain-models) poświęcony programowaniu według wzorca Domain Model. Jeden z wniosków płynących z tego artykułu, to rezygnacja z wyrzucania wyjątków poza Domain Model, czy też szerzej, poza całą warstwę logiki biznesowej. Jest to zdecydowanie inne podejście od większości promowanych reguł budowania aplikacji złożonych z warstw, gdzie zazwyczaj zaleca się, aby w danej warstwie zdefiniować własny wyjątek i wszystkie przez nas generowane lub wyłapane wyjątki z warstw niższych (zależnych) opakowywać wyjątkiem warstwy i przekazywać dalej.

Rozwiązanie z bloga Dahan'a zdecydowanie bardziej mi się podoba niż przekazywanie wyjątków z warstwy biznesowej do warstwy prezentacji. Jedyne moje zastrzeżenia budzi klasa `FailureEvents`, służąca do obsługi błędów płynący z logiki biznesowej. Razi mnie to, iż jest ona statyczna. Osobiście wydaje mi się, przynajmniej jeśli mówimy o programowaniu w ramach Windows Forms, iż znacznie lepiej byłoby wprowadzić fasadę umożliwiającą wywoływanie klas Domain Model i zawierającą niestatyczne zdarzenia obsługi sytuacji błędnych.

Polecam również komentarz [Ayende Rahien'a](http://www.ayende.com/Blog/archive/2008/03/24/Re-How-to-create-fully-encapsulated-Domain-Models.aspx) twórcy [Rhino.Mocks](https://github.com/hibernating-rhinos/rhino-mocks), który między innymi proponuje zastąpienie zdarzeń metodami zwrotnymi (tutaj w pełni zgadzam się z Ayende).