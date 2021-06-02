---
title: Pobranie projektów rozwiązania czyli Regex w akcji
date: 2009-10-08T00:17:00+02:00
languageCode: pl-pl
description: Pobranie przy pomocy PowerShell i wyrażeń regularnych regex projektów z pliku rozwiązania sln środowiska Visual Studio
categories:
  - Development
  - Tools
tags:
  - PowerShell
  - Regular Expressions
  - Visual Studio
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/10/08/pobranie-projekt-w-rozwi-zania-czyli-regex-w-akcji.aspx
disqus_disable: true
---

Asumpt do poniższego rozwiązania dostarczył skrypt PowerShell, który kompiluje projekty pewnego mojego rozwiązania, i gdzie zapałałem chęcią automatycznego uaktualnienia numeru wersji we wszystkich plikach **AssemblyInfo.cs**. Ale gdzież są te wszystkie pliki? No w projektach…

Najprostszy sposób dotarcia do projektów to odczytanie pliku **.sln**. Hm… ale to oznacza analizowanie zawartości. Z pomocą przyszły wyrażenia regularne oraz świadomość istnienia jasno określonej struktury pliku **.sln**.

```powershell
Project\("\{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC\}"\) = "(?<projectName>[^"]+)", "(?<projectFolder>[^"]+)", "(?<projectGuid>[^"]+)"\nEndProject
```

Oczywiście kilka słów wyjaśnienia. Po pierwsze wszystkie projekty rozwiązania jak i również foldery (**Solution Folders**) są umieszczane w sekcji zaczynającej się od słów `Project` a kończącej się na `EndProject`. Elementy będące projektami są oznaczone odpowiednim identyfikatorem Guid (konkretnie `{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}`). Następnie w cudzysłowach danej sekcji mamy kolejno nazwę projektu, ścieżkę względną do projektu oraz unikalny identyfikator Guid przypisany do danego projektu. Dla ułatwienia pobierania informacji zastosowałem grupy nazwane. Czas na prosty przykład użycia:

```powershell
$pattern =
    "Project\(`"\{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC\}`"\) = " +
    "`"(?<projectName>[^`"]+)`", " +
    "`"(?<projectFolder>[^`"]+)`", " +
    "`"(?<projectGuid>[^`"]+)`"\r\nEndProject"
$regex = [regex] $pattern
$solution = [System.IO.File]::ReadAllText('d:\Solution\MySolution.sln')
$regex.Matches($solution) | ForEach-Object {
    Write-Host $_.Groups['projectName']
    Write-Host $_.Groups['projectFolder']
    Write-Host $_.Groups['projectGuid']
    Write-Host
}
```

Ze względu na składnię PowerShell wzorzec wyrażenia regularnego został nieco zmodyfikowany.

PS. Oczywiście można również zastosować prostsze wyrażenie regularne:

```powershell
\{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC\}"\) = "(?<projectName>[^"]+)", "(?<projectFolder>[^"]+)", "(?<projectGuid>[^"]+)
```

lub też bardziej odporne na potencjalne błędy (czy aby na pewno takie będą?):

```powershell
\{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC\}"\)\s*=\s*"(?<projectName>[^"]+)",\s*"(?<projectFolder>[^"]+)",\s*"(?<projectGuid>[^"]+)
```

Ale to już sztuka dla sztuki.