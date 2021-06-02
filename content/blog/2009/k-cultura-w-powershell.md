---
title: (K)Cultura w PowerShell
date: 2009-12-18T23:19:00+01:00
languageCode: pl-pl
description: Polskie znaki narodowe i kodowanie UTF8 plików w PowerShell
categories:
  - Development
tags:
  - .NET
  - PowerShell
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/12/18/k-cultura-w-powershell.aspx
disqus_disable: true
---

"Bo kultura tu naprawdę jest, świadczy o tym nasz wspaniały Dom Kultury" śpiewał w 1988 roku w Jarocinie zespół "Zielone Żabki". Ktoś pamięta? Dziś też będzie o kulturze, ale przez literę c czyli o Culture. Tekst zaś dotyczył będzie tak prozaicznej kwestii jak polskie znaki narodowe.

Zacznijmy od początku. Utwórzmy, np. w Notatniku, plik w formacie CSV zawierający nazwy ptaków z polskimi znakami narodowymi:

```csv
    id,nazwa
    1,Gżegżółka
    2,Żuraw
    3,Łabędź
```

I spróbujmy go wczytać korzystając ze standardowego polecenia PowerShell:

```powershell
    Import-Csv 'c:\ptaki.csv'
```

Jak sie można domyśleć (wstęp to sugerował) to, co ujrzą nasze oczy nie będzie ładne. Zamiast polskich znaków będą mniej lub bardziej nieokreślone śmieci (zależy gdzie uruchomimy nasz kod) zgodne z UTF8. Skąd wiemy, że z UTF8? Empirycznie będzie można to sprawdzić za chwil kilka.

Wstępnie problem został więc rozwiązany. Jak natomiast zmusić PowerShell do wczytania pliku CSV zgodnie z polską stroną kodową? Okazuje się, iż najłatwiej jest skorzystać z polecenia `Get-Content`, które domyślnie bierze pod uwagę ustawienia regionalne systemu operacyjnego komputera.

```powershell
    Get-Content 'c:\ptaki.csv' | ConvertFrom-Csv
```

Dzięki zaś poleceniu `ConvertFrom-Csv` oraz skorzystania z potoku możemy cieszyć się poprawnością wyświetlania:

```shell
    id  nazwa
    --  -----
    1   Gżegżółka
    2   Żuraw
    3   Łabędź
```

Polecenie `Get-Content` posiada również, co ciekawe, parametr `Encoding`, który pozwala ustawić kilka możliwych stron kodowych. Nie są to powalające ilości, ale zawsze coś. W przypadku ustawienia kodowania na UTF8 otrzymamy dane w postaci identycznej jak w przypadku polecenia `Import-Csv`. Aby natomiast uzyskać efekt identyczny jak w przypadku wywołania tego polecenia bez strony kodowej należy parametr `Encoding` ustawić na wartość `String`.

```powershell
    Get-Content 'c:\ptaki.csv' -Encoding String | ConvertFrom-Csv
```

Co zaś w przypadku kiedy możliwości lingwistyczne standardowych poleceń PowerShell nas nie usatysfakcjonują? Zawsze możemy skorzystać z metod klas platformy .NET:

```powershell
    [System.IO.File]::ReadAllText(
        'c:\ptaki.csv',
        [System.Text.Encoding]::GetEncoding(1250)) | ConvertFrom-Csv
```

I na koniec inny, niż skorzystanie z Notatnika, wariant tworzenia plików CSV:

```powershell
    Set-Content 'c:\ptaki.csv' 'id,nazwa'
    Add-Content 'c:\ptaki.csv' '1,Gżegżółka'
    Add-Content 'c:\ptaki.csv' '2,Żuraw'
    Add-Content 'c:\ptaki.csv' '3,Łabędź'
```

Oczywiście w tych poleceniach również możemy skorzystać z parametru `Encoding`.