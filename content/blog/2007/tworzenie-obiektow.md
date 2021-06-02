---
title: Tworzenie obiektów
date: 2007-07-04T13:55:00+02:00
lastmod: 2018-09-17T14:20:00+02:00
languageCode: pl-pl
description: Porównianie sposobów tworzenia obiektów poprzez Inversion of Control, Push. Don't Pull oraz Dependency Injection
catgories:
  - Development
tags:
  - .NET
  - .NET Compact Framework
  - Design patterns
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/07/04/tworzenie-obiekt-w.aspx
disqus_disable: true
---

W powyższej kwestii pojawiło się od czasu słowa kluczowego `new` trochę nowych metod i związanych z tym pojęć. W poszukiwaniu optymalnego kodu warto zwrócić uwagę na następujące pojęcia:

* Inversion of Control - obiekt nie tworzy samodzielnie żadnych wymaganych przez siebie instancji klas. Zamiast tego pobiera je z zewnętrznych zasobów;
* Push. Don't Pull - obiekt nie tworzy samodzielnie żadnych wymaganych przez siebie instancji klas. Zamiast tego są mu one przekazywane jako parametry konstruktora lub właściwości;
* Dependency Injection - tworzenie instancji wymaganych klas zleca się obiektowi (kontenerowi) zewnętrznemu, który zna zależności pomiędzy klasami.

Zastosowanie Inversion of Control powoduje, iż możemy całą logikę tworzenia klas skupić w jednym lub kilku miejscach w programie, korzystając przy tym z metod fabryki, fabryk abstrakcyjnych czy też z wzorca lokalizacji usług (Service Locator), który może być, w zależności od sposobu implementacji, niejako fabryką fabryk abstrakcyjnych. Dzięki takiemu podejściu łatwiejsze jest zarządzanie tworzeniem instancji klas, ich powtórnym wykorzystaniem czy też podmianą implementacji w zależności od potrzeb.

Gdzie tylko jest to możliwe należy zaś korzystać z Push. Don't Pull. Obiekt nie powinien wiedzieć za dużo o swoim otoczeniu. To po pierwsze. A po drugie taki sposób postępowania ułatwia testowanie klas. Z doświadczenia mogę powiedzieć, iż to podejście staje naturalne i oczywiste jeśli korzysta się z klas testujących. Częstą praktyką jest również łączenie w ramach klasy Inversion of Control z Push. Don't Pull poprzez deklarację dwóch konstruktorów. Pierwszy umożliwia przekazanie jako parametrów wszystkich wymaganych obiektów i ten konstruktor może być wykorzystywany do testów. Drugi konstruktor ustawia te samo zmienne klasy co pierwszy, ale pobiera instancje wymaganych klas z zasobów zewnętrznych.

Korzystanie z Dependency Injection wymaga zaś od nas wsparcia się zewnętrznymi narzędziami. Wśród nich wymienić można [StructureMap](https://jeremydmiller.com/category/structuremap/), [Spring.NET](http://www.springframework.net), czy też [Windsor Container](http://www.castleproject.org). Programiści urządzeń mobilnych (platforma .NET CF) mogą skorzystać z możliwości Mobile ObjectBuilder z [Mobile Client Software Factory](http://www.codeplex.com/smartclient). Zależności pomiędzy klasami definiuje się zazwyczaj w postaci konfiguracji w plikach w formacie XML lub korzystając z atrybutów. Oczywiście taki sposób działania wymaga pewnych zasobów komputera co może mieć krytyczne znaczenie zwłaszcza dla aplikacji mobilnych.