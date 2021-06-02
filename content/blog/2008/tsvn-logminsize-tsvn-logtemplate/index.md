---
title: tsvn:logminsize, tsvn:logtemplate
date: 2008-02-14T16:00:00+01:00
languageCode: pl-pl
description: Wymagana minimalna liczba znaków i szablon komunikatu przy zatwierdzaniu zmian w repozytorium kodu Subversion przy korzystaniu z klienta TortoiseSVN
categories:
  - Tools
tags:
  - Subversion
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/02/14/tsvn-logminsize-tsvn-logtemplate.aspx
disqus_disable: true
---

Ostatni [wpis]({{< ref "hook-scripts-w-csharp" >}}) dotyczył wykorzystania skryptów przechwytujących do zablokowania dokonywania modyfikacji w ramach repozytorium Subversion jeśli nie zostanie podana informacja opisująca przyczynę zmian. Opisywana metoda działa na poziomie repozytorium. Jeśli jako klienta Subversion wykorzystujemy [TortoiseSVN](http://tortoisesvn.tigris.org/), możemy zamiast tego wykorzystać możliwości, jakie oferują specjalne właściwości, które można przypisać do katalogu objętego kontrolą SVN. Ponieważ są to rozszerzenia TortoiseSVN, które podobnie jak właściwości Subversion mogą być przez użytkownika modyfikowane i usuwane, nie są one mocnym zabezpieczeniem. Ale czasem są oczywiście w zupełności wystarczające. Te rozszerzenia to między innymi:

* **tsvn:logminsize** - który definiuje minimalną dozwoloną długość komunikatu;
* **tsvn:logtemplate** - szablon komunikatu, który może być przez użytkownika uzupełniony lub zmieniony.

Aby ustawić te parametry wybieramy główny katalog danych pobranych z repozytorium (**local working copy**). Klikamy prawym klawiszem, wybieramy polecenie **Właściwości** a następnie zakładkę **Subversion**.

{{< figure src="images/tsvn_log.png" title="Ustawienie ograniczeń dotyczących informacji o zmianach w repozytorium" >}}

Dalej klikamy przycisk **Properties...** -> **Add...** (**Dodaj...**) i jesteśmy w domu. Z rozwijalnej listy wybieramy interesujące nas parametry i ustawiamy ich wartości.