---
title: F(a) = (a + k) mod n
date: 2007-09-25T15:45:00+02:00
languageCode: pl-pl
description: Wzór na znalezienie wartości ze zbioru n elementów, gdzie a stanowi punkt odniesienia (aktualna wartość), a k jest przesunięciem
categories:
  - Development
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/09/25/f-a-a-k-mod-n.aspx
disqus_disable: true
---

Powyższy wzór umożliwia nam znalezienie nowej wartości ze zbioru **n** elementów, gdzie **a** stanowi punkt odniesienia (aktualna wartość), a **k** jest przesunięciem. Dzięki temu algorytmowi możemy potraktować zbiór jako bufor kołowy, dzięki czemu nie musimy sprawdzać wartości brzegowych. Oznacza to, iż po osiągnięciu ostatniego elementu zbioru, zostanie wybrany element pierwszy.

Dobry przykład zastosowania to przeglądanie na ekranie danych szczegółowych produktu. Załóżmy, iż chcemy umożliwić użytkownikowi wyświetlenie jednocześnie tylko jednej  z poniższych wartości:

* Kod produktu;
* Cena produktu;
* Stawka VAT;
* Ilość na stanie magazynu.

Wyboru dokonujemy klawiszem:

* Strzałka w prawo dla przesunięć od kodu produktu w stronę ilości;
* Strzałka w lewo dla przesunięć od ilości do kodu produktu.

Dla tych kombinacji otrzymujemy następujące wzory:

* Strzałka w prawo: **mode = (mode + 1) % 4**;
* Strzałka w lewo: **mode = (mode + 3) % 4**.

Zmienna **mode** oznacza wartość aktualną.