---
title: Zacznij od nowej strony
date: 2009-02-25T15:26:00+01:00
lastmod: 2018-09-17T14:10:00+02:00
languageCode: pl-pl
description: Tips and tricks. Rozwiązanie problemu wyświetlenia w Reporting Services podraportu wiele razy zaczynając za każdym razem od nowej strony
categories:
  - Databases
tags:
  - Reporting Services
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/02/25/zacznij-od-nowej-strony.aspx
disqus_disable: true
---

Wyświetlanie w Reporting Services podraportu w ramach raportu nie jest trudne. Schody zaczynają się jednak wtedy, kiedy zapragniemy dany podraport wyświetlić wiele razy zaczynając za każdym razem od nowej strony. To już nie jest takie łatwe. Istnieje wiele zasad co i w jakiej kolejności jest wykonywane w czasie przygotowania strony do  [wyświetlenia](https://technet.microsoft.com/en-us/library/bb677570.aspx). Istnieje również wiele pomysłów jak zlikwidować problemy z niemożnością rozpoczynania podraportów na nowej stronie. Najzabawniejsze rozwiązanie jakie widziałem to takie, w którym autor przekonywał, iż podraport należy po prostu przekopiować do raportu…

Moje rozwiązanie jest proste i skuteczne – zawsze działa. W ramach raportu umieszczam kontrolkę **List**, do której wstawiam kontrolkę **Subreport**. I teraz najważniejsze. Poniżej kontrolki **Subreport**, ale ciągle w ramach **List** umieszczam kontrolkę **Rectangle**. Ustalam wysokość bramowania na 0,1 cm aby nie burzyć wyglądu. Na koniec ustawiam właściwość **PageBreakAtEnd** na **True**. I to wystarczy. Życie staje się prostsze.