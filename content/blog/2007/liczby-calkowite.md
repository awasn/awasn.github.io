---
title: Liczby całkowite
date: 2007-05-10T12:44:00+02:00
languageCode: pl-pl
description: Liczby całkowite przechowywane w bazie danych jako wartość tekstowa
categories:
  - Development
  - Databases
tags:
  - .NET
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/05/10/liczby-ca-kowite.aspx
disqus_disable: true
---

Ostatnimi czasy byłem granatem oderwany od .NET i przerzucony na front języka AMBASIC, Symfonii Handel i jej raportów (konkretnie chodziło o integrację z systemem [RouteSpider](http://www.soft-tronik.com/), którym się obecnie między innymi zajmuję).

Analizując tabele Symfonii mogłem zaobserwować, jak twórcy tego produktu musieli się zmierzyć z problemem wydajności. Część pól występuje bowiem w strukturach podwójnie. Jako pole tekstowe oraz pole numeryczne, przy czym nazwa pola numerycznego jest niczym więcej jak nazwą pola tekstowego z dodaną na końcu literką i. Każdy, kto kiedykolwiek musiał usprawniać porównania pewnie wie, jak czasochłonne są to operacje w przypadku łańcuchów znaków. Ocena zgodności dwóch liczb 32-bitowych zapisanych w postaci tekstu jest do kilkuset razy wolniejsze (sic!), niż ta sama operacja wykonana na zmiennych numerycznych (znaczenie ma oczywiście ilość cyfr).

Swoją drogą jest to również ważki przyczynek do twierdzenia, iż klucze główne powinny być w bazach danych tak małe, jak to jest możliwe, co przełożyć można na ludzki język, iż należy stosować w tej roli tylko i wyłącznie liczby całkowite. Nawet w tabelach, które odpowiadają za relacje wiele-do-wielu. Jeśli ktoś lubi stosować w tej roli identyfikatory `Guid`, to powinien mieć świadomość, iż porównanie w środowisku .NET dwóch zmiennych tego typu wymaga w najgorszym przypadku wykonania 11 operacji na liczbach typu `Int32`, `Int16` oraz `Byte`.

Zastanawiające jest również czemu z tabel Symfonii nie zniknęły owe nadmiarowe pole. Czy chodziło o istniejące już raporty, czy może po prostu tego co działa się nie rusza...