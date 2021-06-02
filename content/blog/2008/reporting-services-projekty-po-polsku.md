---
title: Reporting Services - projekt(y) po polsku?
date: 2008-12-16T15:53:00+01:00
languageCode: pl-pl
description: Zdrowie psychiczne jest ważne dlatego prędzej czy później raporty biznesowe i procedury składowane zwracające dla nich dane będziesz nazywał po polsku
categories:
  - Development
  - Databases
tags:
  - Reporting Services
  - SQL Server
  - Visual Studio
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/12/16/reporting-services-projekt-y-po-polsku.aspx
disqus_disable: true
---

Kiedy ponad rok temu zaczynałem budowanie raportów w MS SQL Server 2005 Reporting Services podszedłem do tematu jak rasowy programista. Całe nazewnictwo miało być po angielsku.

## Etap pierwszy

Założyłem nowy projekt, dodałem do repozytorium (usługi raportowe były nowe, nowiuśkie, bez jakiegokolwiek raportu) i... hm.... Pierwsza niespodzianka to siermiężne warunki przygotowywania zapytań w Visual Studio. Nie będę się rozpisywał. Budowanie zapytań SQL w tym środowisku ma swoje ograniczenia (ma się ochotę przejść na ciemną stronę mocy).

## Etap drugi

Chciał nie chciał, ale powstał skorelowany z raportami drugi projekt przeznaczony dla Microsoft SQL Server Management Studio. Teraz życie uległo poprawie. W Management Studio powstawało zapytanie, lub zapytania jeśli raport miał zawierać parametry w postaci rozwijalnych list, i testy poprawności zwracanych danych. W Visual Studio zaś wygląd raportu. Zapytanie było przekopiowywane (Ctrl+C, Ctrl+V).

## Etap trzeci

Raportów przybywa. Nazewnictwo specjalistyczne doprowadza przy tłumaczeniach do szału. W ramach samej usługi dostępnej poprzez stronę WWW raporty te występują pod polskimi nazwami, co jest słuszne - w końcu pracuję w polskiej firmie. Problemy zaczynają sprawiać łączenia raportów (budowanie odnośników w jednych raportach do drugich). Zaczynam się irytować.

## Etap czwarty

Nazwy wszystkich plików zawierających zapytania SQL oraz definicje raportów zostały przetłumaczone na polski. Powrót do macierzy. Trochę to jeszcze drażni ale tak jest zdecydowanie lepiej. Nie ma oznak paniki. Łatwo znaleźć raport, o który właściciel biznesowy się dopytuje.

W miarę powstawania kolejnych raportów coraz więcej osób próbuje wykorzystać platformę raportową w celu ułatwienia sobie codziennej pracy. Ponieważ zapytania są budowane na relacyjnej bazie danych ich rozmiar staje się coraz większy i większy.... aż pewnego dnia Visual Studio mówi stop! Przekroczyłem limit 32 KB znaków w zapytaniu. Coś podobnego. Usunięcie komentarzy pomogło.

Zdarza się również, iż zmieni się logika biznesowa (czytaj znaczenie pól w tabelach) albo zleceniodawca raportu ma fanaberie i trzeba któreś z pól trochę inaczej przeliczyć. Zmiana zapytania, przekopiowanie zapytania do raportu, połączenie się poprzez RDP z serwerem, wgranie raportu.... Ta ścieżka zaczyna być zbyt nużąca.

## Etap piąty

Kocham procedury składowane. Wszystko składuję teraz w nich (no bez przesady. Własne funkcje i procedury do pomocniczych obliczeń używałem od samego początku). Całe zapytania. Raport wywołuje procedurę i tyle. Żadnego kopiowania. Jeśli pojawia się zmiana obliczania pól, które nie skutkuje modyfikacją raportu to wystarczy zmienić tylko procedurę. Bez konieczności modyfikowania zapytania w definicji raportu.

Oczywiście procedury mają polskie nazwy. I tak zostanie.