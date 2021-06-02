---
title: Rozwiązanie mobilne
date: 2008-12-11T12:45:00+01:00
lastmod: 2018-09-17T13:19:00+02:00
languageCode: pl-pl
description: Opis projektu sprzedaży mobilnej przeznaczonego dla profesjonalnych urządzeń mobilnych. Platforma .NET i .NET Compact Framework 
categories:
  - Development
  - Databases
tags:
  - .NET
  - .NET Compact Framework
  - ASP.NET
  - Design patterns
  - IIS
  - Mobile
  - ReSharper
  - Software testing
  - SQL Server
  - Subversion
  - Virtualization
  - Visual Studio
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/12/11/rozwi-zanie-mobilne.aspx
disqus_disable: true
---

Kilka miesięcy temu stałem się szczęśliwym posiadaczem projektu na przygotowanie kompletnego rozwiązania z zakresu szeroko pojętej sprzedaży dla "profesjonalnych" urządzeń mobilnych, czyli komputerów odpornych na niskie temperatury i kroploszczelne. Systemy operacyjne, które znalazły się w zasięgu projektu to:

* Microsoft Windows for Pocket PC 2003, Windows Mobile 2003;
* Windows Mobile 5.0 for Pocket PC, Windows Mobile 5.0 for Pocket PC Phone Edition;
* Windows Mobile 6 Classic, Windows Mobile 6 Professional.

Pod uwagę należało też brać  możliwość pojawienia  się w przyszłości konieczności działania programu w ramach Windows CE .NET. Wstępnie zostały natomiast wykluczone urządzenia typu Smartphone.

Całe rozwiązanie miało być stworzone przeze mnie. Miałem więc pełnić rolę analityka, architekta, programisty czy testera. Do mnie należał wybór technologii i architektury. Od strony programisty trudno o lepszą okazję. Z drugiej strony brak możliwość konsultacji czy weryfikacji pomysłów może być frustrujący.

Główną wadą projektu były krótkie terminy na dostarczenie nawet nie prototypu, ale wręcz działającego systemu. Jeśli chodzi o brak czasu to miałem nawet pewną koncepcję jak to rozwiązać. Przyszło mi bowiem do głowy skorzystanie ze Szczególnej teorii względności Alberta Einsteina, która mówi, iż w układzie własnym czas biegnie wolniej, co jest odczuwalne zwłaszcza kiedy zbliżymy się do prędkości światła. Niestety problemy ze znalezieniem układu odniesienia i punktu podparcia Ziemi spowodowały, iż pomysł spowolnienia czasu poprzez danie kopa naszej planecie spalił na  panewce. Trzeba było więc wziąć się szybciutko do roboty.

Nad wyborem platformy programistycznej nie było co się zastanawiać. W grę wchodził tylko .NET Compact Framework wraz z Visual Studio w wersji minimum Professional. Wymaganie co do wersji środowiska programistycznego spowodowane było brakiem w wersji Express i Standard możliwości tworzenia projektów dla .NET CF.

Po zapoznaniu się z założeniami stwierdziłem, iż projektowane rozwiązanie będzie się składać z następujących elementów:

* Aplikacji mobilnej współpracującej z drukarką fiskalną;
* Serwera komunikacji odpowiedzialnego za wymianę danych;
* Bibliotek konwertujących umożliwiających korzystanie przez serwer komunikacji z różnych źródeł danych (systemów sprzedaży).

Podstawą projektową dla oprogramowania mobilnego miał być wzorzec Model-View-Presenter. Przy tworzeniu tej części rozwiązania należało uwzględnić ograniczenia urządzeń przenośnych związanych z mocą obliczeniową, dostępnymi zasobami pamięci czy możliwościami komunikacyjnymi. Konkretna konfiguracja komputera zależeć miała od wymagań potencjalnego użytkownika. Aplikacja musiała być bardzo szybka. Czasy oczekiwania na wyświetlenie dowolnego formularza wypełnionego danymi nie  powinny przekroczyć pół sekundy. Pewnym ułatwieniem był fakt, iż program nie musiał blokować korzystania z innych aplikacji systemu operacyjnego. Z drukarką fiskalną nie miałem problemów ponieważ sterownik do niej był już wcześniej przeze mnie stworzony (ktoś jeszcze korzysta z RS232? Włącza linie etc...?).

Do wymiany danych pomiędzy aplikacją mobilną a systemami klienta postanowiłem wykorzystać możliwości serwera [Internet Information Services](http://iis.net) oraz środowiska ASP.NET. Być może niektórzy pamiętają, iż w pierwszej połowie roku podobne rozwiązanie zacząłem opisywać na blogu. Ukazały się dwa wpisy: pierwszy poświęcony był [ogólnym rozważaniom dotyczącym komunikacji]({{< ref "http-wlasny-serwer-komunikacji-cz1-wstep" >}}) drugi zaś dotyczył [konfiguracji środowiska programistycznego]({{< ref "http-wlasny-serwer-komunikacji-cz2-srodowisko-programistyczne" >}}). Niestety powstające rozwiązanie mobilne do minimum ograniczyło dostępny czas i, jeśli chodzi o wpisy, na blogu powiało u mnie posuchą. Mam jednak nadzieję, iż niedługo pojawią się kolejne części opisujące komunikację z wykorzystaniem IIS.

Biblioteki konwertujące jak dotąd powstały dwie. Jedna do bardzo podstawowej integracji z SAP z wykorzystaniem  dedykowanych plików wymiany oraz druga umożliwiająca współpracę z systemem sprzedaży Subiekt GT firmy InsERT.

Stworzenie wszystkich elementów samodzielnie i od podstaw dały mi  możliwość sprawdzenia w boju niektórych koncepcji i nowinek programistycznych. O części z nich będzie w następnych wpisach. Poza tym jakaż satysfakcja. Nie do przecenienia.

Jeśli chodzi o narzędzia to korzystałem między innymi z:

* Visual Studio 2005 Professional (do pierwszej bety aplikacji mobilnej) a następnie z Visual Studio 2008 Professional;
* Wspomagacz kodu [ReSharper](http://www.jetbrains.com/resharper). Bez tej wtyczki nie wyobrażam sobie wydajnej pracy. Darmową licencje na  program [ReSharper]({{< ref "resharper-ostrzega-possible-nullreferenceexception" >}}) (który jest płatny) otrzymałem za prezentacje na spotkaniach [Warszawskiej Grupy .NET](http://wg.net.pl) oraz za udzielanie się na zine.net;
* [Subversion](http://subversion.apache.org) jako repozytorium danych. [Repozytorium plikowe]({{< ref "subversion" >}}) składowane na paluchu USB (nota bene zasponsorowanym swego czasu przez [Red Gate](http://www.red-gate.com) za prezentacje na [Warszawskiej * Grupie .NET](http://wg.net.pl)). Klienci SVN używani przeze mnie to [TortoiseSVN](http://tortoisesvn.tigris.org) oraz wtyczka do  Visual Studio [AnkhSVN](http://ankhsvn.open.collab.net). Krótka uwaga co do AnkhSVN - odkąd projekt został przejęty przez [CollabNet](http://www.collab.net) wtyczka jest godna polecenia i instalacji;
* [NUnit](http://nunit.org) oraz [Rhino.Mocks](https://github.com/hibernating-rhinos/rhino-mocks) jako elementy środowiska testowego. W czasie pracy w Visual Studio testy wywoływałem głównie korzystając  z ReSharper'a;
* [Microsoft Virtual PC](https://www.microsoft.com/en-us/download/details.aspx?id=3702) do testowania komunikacji oraz wersji czasowych aplikacji, z którymi tworzone rozwiązanie musiało  się integrować;
* Microsoft SQL Server 2005 Compact  Edition jako miejsce składowania danych po stronie aplikacji mobilnej;
* [NLog](http://nlog-project.org) jako biblioteka umożliwiająca rejestrowanie  zdarzeń w ramach platformy .NET i .NET Compact Framework.

Na dzień dzisiejszy całe rozwiązanie działa i jest już używane. W tak zwanym międzyczasie zrefaktoryzowałem projekt w części mobilnej i wprowadziłem kilka nowości. Mam też nadzieję, iż system ten będzie dalej rozwijany. Tak się przynajmniej na razie zapowiada.

Wszystkich, którzy mają jakiekolwiek uwagi, propozycje ;-) etc.  zapraszam do kontaktu i komentarzy. I do czytania następnych odcinków.