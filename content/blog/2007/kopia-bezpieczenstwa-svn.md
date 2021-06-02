---
title: Kopia bezpieczeństwa SVN
date: 2007-10-19T15:41:00+02:00
languageCode: pl-pl
description: Skrypt bat do tworzenia kopii bezpieczeństwa repozytorium kodu Subversion
categories:
  - Tools
tags:
  - Subversion
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/10/19/kopia-bezpiecze-stwa-svn.aspx
disqus_disable: true
---

Repozytoria Subversion, zgodnie z dokumentacją, mogą być archiwizowane w locie. Może jednak się zdarzyć, iż wykonywane w tak zwanym między czasie przez użytkowników operacje spowodują, iż otrzymana kopia, mimo swej poprawności będzie logicznie niespójna. Ze względu bowiem na czas trwania archiwizacji może dojść do sytuacji, iż w repozytorium zostaną zatwierdzone kolejne transakcje i nasza kopia części nowych danych nie uwzględni. Z tego też powodu zaleca się wcześniejsze przygotowanie repozytorium SVN do operacji archiwizacji danych. W tym celu możemy wykorzystać będące częścią serwera narzędzie jakim jest **svnadmin**, które kopiuje w określone parametrem miejsce zawartość całego repozytorium w taki sposób, aby zachować jego spójność.

```dos
@echo;
@echo; Kopia bezpieczeństwa repozytorium Subversion
@echo;

SET wczoraj=d:\svn-repozytorium\kopia\wczoraj
SET dzisiaj=d:\svn-repozytorium\kopia\dzisiaj

rd /s /q %wczoraj%
move %dzisiaj% %wczoraj%
mkdir %dzisiaj%

svnadmin hotcopy d:\svn-repozytorium\repozytorium\programowanie %dzisiaj%\programowanie
```

Powyższy skrypt **.bat** stosowany był przeze mnie do przygotowania repozytorium o wiele znaczącej nazwie **programowanie**. Jeśli przygotowujemy do archiwizacji repozytorium typu Berkeley DB możemy na końcu wywołania **svnadmin** dodać parametr **--clean-logs**, które wyczyści nieużywane logi.

Skojarzenie z takim skryptem **Zaplanowanego zadania** pozwoli nam zautomatyzować cały proces. Oczywiście wystarczy, kiedy program archiwizujący będzie analizował jedynie katalog **d:\svn-repozytorium\kopia\dzisiaj**.

PS. Zgadzam się, iż tworzenie dwóch katalogów może być oznaką manii prześladowczej...