---
title: TDD versus BDD
date: 2009-03-17T12:02:00+01:00
languageCode: pl-pl
description: Różnica pomiędzy Test Dirven Driven Development (TDD) a Behavior Driven Development (BDD) w kilku zdaniach
categories:
  - Development
tags:
  - Software testing
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/03/17/tdd-versus-bdd.aspx
disqus_disable: true
---

Testy, testy, testy. Najpierw. Przed właściwym kodem. A może po? Co testować? Wszystko? Jeśli tworzymy zbiór publicznych typów (interfejs pakietu) to testowane powinno być jak najbardziej wszystko i to szczegółowo. To jest Test Dirven Driven Development (TDD). Część publiczna jest naszym kontraktem informującym jaki będzie  rezultat operacji przy określonych parametrach. Niestety w chwili obecnej możliwości języków programowania są ograniczone jeśli chodzi o precyzyjne definiowanie dozwolonych wartości przekazywanych do metod i właściwości. Dlatego też musimy się posiłkować wyjątkami oraz dokumentacją do poszczególnych elementów kontraktu.

Ale jeśli powstające metody i właściwości są w typach wewnętrznych względem przestrzeni nazw, a co za tym idzie pakietu? Czy mam sprowadzać swoje życie do absurdu? Jest oczywiste, że nie będę sprawdzał w każdej metodzie parametrów i wyrzucał w przypadku wadliwych danych wyjątków. Na takim poziomie bardziej interesuje mnie poprawność wykonania ścieżki logicznej, zachowania. Moje intencje oznaczam wówczas w kodzie metodami `Debug.Assert`. W dalszym ciągu dostarczany przeze mnie kod stanowi kontrakt, ale nie sprawdzam poprawności wszystkich parametrów, nie wyrzucam wyjątków. Taki zaś jest Behavior Driven Development (BDD).