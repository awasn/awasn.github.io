---
title: Dobre praktyki w projektowaniu aplikacji mobilnych
date: 2007-04-23T14:03:00+02:00
languageCode: pl-pl
description: Prezentacja dotycząca wzorców projektowych na spotkaniu Warszawskiej Grupy .NET
categories:
  - Development
  - Events
tags:
  - .NET Compact Framework
  - Design patterns
  - Meetings
  - Mobile
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/04/23/dobre-praktyki-w-projektowaniu-aplikacji-mobilnych.aspx
disqus_disable: true
---

W ostatnią sobotę (21.04.2007) na VI spotkaniu Warszawskiej Grupy .NET miałem przyjemność wystąpić z prezentacją dotyczącą wzorców projektowych, które warto zastosować programując urządzenia mobilne. Sam plik zawierający slajdy to oczywiście za mało, więc poniżej kilka dodatkowych opisów.

## Service Locator, slajdy 18 i 19

Klasa `ObjectLocator` oparta o wzorzec `ServiceLocator` została, ze względu na chęć umieszczenia wszystkiego na slajdzie, właściwościach pozbawiona możliwości nadawania nowych wartości. Rzeczywista klasa powinna taką możliwość udostępniać - chociażby w celu umożliwienia napisania klas testujących. Oczywiście można pozostawić właściwości jako tylko do odczytu i dodać dodatkowy konstruktor (np. `internal`) umożliwiający przypisanie odpowiednich wartości początkowych.

## Przykładowa implementacja interfejsu prezentera, slajd 35

W czasie dyskusji podczas prezentacji padła sugestia, iż klasa prezentera, w tym wypadku `MainMenuPresenter`, powinna być zarejestrowana w systemie, podobnie jak widoki ze slajdu 32, poprzez interfejsy. Chodzi o sytuację, kiedy jeden użytkownik aplikacji chce mieć w menu głównym powiedzmy sześć poleceń menu, a inny tylko pięć. Wtedy faktycznie, tego typu działanie byłoby bardziej wskazane. Pomysł ten jednak nie rozwiązuje przypadku, kiedy w aplikacji mamy np. formularz wyświetlający listę produktów w kilku miejscach (przypadkach użycia) programu i jeden z użytkowników zapragnie, aby w konkretnej sytuacji była np. inna kolejność kolumn etc. W tego typu sytuacjach lepiej pozostać przy sposobie zaproponowanym na slajdzie, wprowadzając natomiast klasy potomne dla danych przypadków użycia, czyli klas odpowiadających za logikę przejść pomiędzy ekranami.

Pliki:

* <http://zine.net.pl/files/folders/220/download.aspx> - Prezentacja.