---
title: Mobilne MFC a protokół HTTP
date: 2007-08-31T10:37:00+02:00
languageCode: pl-pl
description: Czyszczenie pamięci podręcznej programu Pocket Internet Explorer aby zwolnić zajęte przez klasy MFC zasoby
categories:
  - Development
tags:
  - Mobile
  - C/C++
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/08/31/mobilne-mfc-a-protok-http.aspx
disqus_disable: true
---

Wakacje nie wakacje, szukanie pracy czy nie, wypadałoby zadbać o swój blog. Dzisiaj trochę z innej beczki (chociaż jak się zastanowić to nie znajduję argumentów czemu z innej niby beczki) ale wciąż w temacie mobilnym. Po to aby nie zapomnieć.

Osoby programujące urządzenia mobilne i korzystające z klas MFC w ramach Microsoft eMbedded Visual C++ i chcące do wymiany danych wykorzystać protokół HTTP natkną się na niemiłą niespodziankę. Każda odebrana odpowiedź z serwera zmniejsza nam ilość dostępnej pamięci na dysku. Jeśli pobierzemy z sieci plik i zapiszemy go na dysku, wolna przestrzeń zostanie zmniejszona dwukrotnie biorąc pod uwagę rozmiar pliku. Ki diabeł?

Problem polega na tym, iż MFC (między innymi klasy `CInternetSession`, `CHttpFile`) do obsługi protokołu HTTP wykorzystuje zasoby programu Pocket Internet Explorer, który odebrane dane przechowuje w pamięci podręcznej. Sprawdzić to można wizualnie w katalogu **Windows\Profiles\guest\Temporary Internet Files\Content.IE5**. Oczywiście możemy z poziomu tego programu zwolnić zajęte zasoby, ale taka filozofia jest mi z gruntu obca ideowo. Jak inaczej do tego podejść?

W ramach wersji systemu Pocket PC 2002 i na części urządzeń wykorzystujących Pocket PC 2003 pewnym rozwiązaniem były odpowiednie wpisy w rejestrach (gałąź **HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\InternetSettings** i podgałęzie **Cache** lub **5.0\Cache** w zależności od wersji systemu). Ale w większości urządzeń sposób ten nie sprawdzał się. Zasobów ubywało.

Obie klasy, to znaczy `CInternetSession` w ramach konstruktora jak i `CHttpFile` w metodzie `OpenRequest`, mogą przyjmować jako jeden ze swoich parametrów flagę `INTERNET_FLAG_DONT_CACHE`. Wspaniale, tylko, co jest w dokumentacji środowiska Microsoft eMbedded Visual C++ wyraźnie napisane, flaga ta nie jest dozwolona w MFC na urządzenia mobilne.

W Microsoft Visual Studio 2005, w której to wersji pojawiła się wreszcie możliwość korzystania z języka C++ przy programowaniu urządzeń mobilnych, biblioteka MFC, dostępna w wersji 8.0, została pozbawiona wymienionych wcześniej klas. Co może dziwić. Na szczęście zanim człowiek się zorientował (refleks szachisty) wraz z Sevice Pack 1, klasy te zostały przywrócone. Ponieważ w dokumentacji nie było żadnych informacji o ograniczeniach dotyczących parametrów przyjmowanych przez metody (a szkoda), stworzyłem prościutką aplikację, która miała potwierdzić kiełkującą w głowie radość. Niestety brutalna rzeczywistość makra `ASSERT` w warunkach zmiennej warunkowej kompilacji `_WIN32_WCE` rozwiała wszelkie nadzieje.

Co zatem więc? Otóż okazuje się, iż rozwiązanie tego problemu jest w zasięgu naszych możliwości programistycznych. Wystarczy skorzystać z dobrodziejstw metod `FindFirstUrlCacheEntry`, `FindNextUrlCacheEntry`, `FindCloseUrlCache` oraz `DeleteUrlCacheEntry`, które umożliwiają nam iterację po elementach pamięci podręcznej (szukamy `COOKIE_CACHE_ENTRY`) oraz usunięcie niepotrzebnych danych. Tyle i aż tyle.

Na pocieszenie zostaje fakt, iż C# w ramach platformy .NET Compact Framework takich ograniczeń nie posiada i możemy z protokołu HTTP korzystać dowoli bez obaw, iż znienacka zabraknie nam zasobów.