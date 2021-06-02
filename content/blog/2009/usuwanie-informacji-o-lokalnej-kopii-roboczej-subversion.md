---
title: Usuwanie informacji o lokalnej kopii roboczej Subversion
date: 2009-04-15T14:07:00+02:00
languageCode: pl-pl
description: Usuwanie informacji o lokalnej kopii roboczej repozytorium kodu Subversion przy pomocy skryptu PowerShell
categories:
  - Development
tags:
  - PowerShell
  - Subversion
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/04/15/usuwanie-informacji-o-lokalnej-kopii-roboczej-subversion.aspx
disqus_disable: true
---

Z przyczyn mniej lub bardziej zrozumiałych koniecznym było, aby usunąć z kilku projektów katalogi zawierające informacje o lokalnej kopii roboczej Subversion *_svn*. Przy braku połączenia z repozytorium kodu ręczne usuwanie to dłubanina i gwarantowana depresja. Z pomocą przyszedł PowerShell. Jak zwykle. Poniżej skrypt usuwający to co trzeba tam gdzie trzeba:

```powershell
Clear-Host
$path = Read-Host "Folder przeznaczony do wyczyszenia z katalogów _svn: "
Get-ChildItem -Path $path -Include _svn -Force -Recurse -Filter FullName |
    Remove-Item -Force -Recurse
Write-Host "Operacja usuwania katalogów _svn zakończona pomyślnie."
```

Krótko, zwięźle i na temat.