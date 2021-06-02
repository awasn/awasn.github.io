---
title: Hook scripts w PowerShell
date: 2009-05-22T21:32:00+02:00
languageCode: pl-pl
description: Skrypty przechwytujące hook scripts w PowerShell dla repozytoriów systemu kontroli wersji Subversion
categories:
  - Development
tags:
  - .NET
  - PowerShell
  - Subversion
series:
  - Subversion hook scripts
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/05/22/hook-scripts-w-powershell.aspx
disqus_disable: true
---

Dawno, dawno temu (choć może nie aż tak dawno) popełniłem [notkę]({{< ref "hook-scripts-w-csharp" >}}) na temat skryptów przechwytujących (**hook scripts**) dla repozytoriów systemu kontroli wersji [Subversion](http://www.open.collab.net/downloads/subversion/). Chodziło o uniemożliwienie zapisania w repozytorium zmian, jeśli nie został podany do nich żaden komentarz wyjaśniający. Proponowany kod wyglądał mniej więcej tak:

```csharp
        private static int Main(string[] args)
        {
            string repositoryPath = args[0];
            string transactionName = args[1];

            var process = new Process();
            process.StartInfo.FileName = "svnlook.exe";
            process.StartInfo.Arguments = string.Format("log -t {0} {1}",
                transactionName, repositoryPath);
            process.StartInfo.UseShellExecute = false;
            process.StartInfo.RedirectStandardOutput = true;
            process.StartInfo.CreateNoWindow = true;
            process.Start();
            string output = process.StandardOutput.ReadToEnd();
            process.WaitForExit();

            var regex = new Regex("[a-zA-Z0-9]");
            if (!regex.IsMatch(output)){
                Console.Error.WriteLine("Brak opisu poczynionych zmian.");
                return 1;
            }

            return 0;
        }
```

Ostatnio pomyślałem czemu by nie wykorzystać do wspomożenia Subversion języka PowerShell. Wykoncypowałem, iż w katalogu, w którym są składowane repozytoria założę folder **scripts**, który będzie zawierał skrypty PowerShell, a do którego będą sięgały pliki wsadowe (**batch files**) wywoływane przez SVN w katalogu **hooks** danego repozytorium.

Pierwsza wersja skryptu sprawdzającego istnienie komentarza do zachowywanych zmian była następująca:

```powershell
        $repositoryPath = $args[0]
        $transactionName = $args[1]

        $message = svnlook.exe log -t $transactionName $repositoryPath
        if($message -notmatch '[a-zA-Z0-9]'){
            Write-Error "Brak opisu poczynionych zmian."
            return 1
        }
```

Okazało się jednak, iż nie funkcjonuje on w zadowalający sposób. Po pierwsze niepoprawnie był zwracany do systemu status zakończenia skryptu (**dos exit code**). Rozwiązań znalezionych po poszukiwaniach w sieci było kilka. Ale najbardziej sensowne, w mojej opinii oczywiście, polegało na skorzystaniu ze specjalnej zmiennej PowerShell `$host`. Drugi problem wiązał się z komunikatem zwrotnym, który chciałem przekazać użytkownikowi próbującemu zatwierdzić zmiany. Aby wiadomość mogła być wyświetlona musi ona zostać wysłana do standardowego strumienia zawierającego błędy. Niestety okazało się, iż skorzystanie z polecenia Write-Error jest niemal tożsame z wyrzuceniem w tym miejscu wyjątku `throw`. Użytkownik, poza komunikatem zwrotnym, ze skryptu otrzymywał również dodatkowe informacje na temat kategorii błędów, miejsca wystąpienia etc. Niedobrze. Na szczęście PowerShell będąc opartym o platformę .NET pozwala bez problemów korzystać z jej bibliotek. Wiadomość dla użytkownika wysłałem więc tak jak w kodzie C# do standardowego strumienia błędów `System.Console.Error`. A oto końcowa wersja skryptu:

```powershell
        $repositoryPath = $args[0]
        $transactionName = $args[1]

        $message = svnlook.exe log -t $transactionName $repositoryPath
        if($message -notmatch '[a-zA-Z0-9]'){
            [System.Console]::Error.WriteLine("Brak opisu poczynionych zmian.")
            $host.SetShouldExit(1)
        }
```

Jak już delikatnie powyżej zasugerowałem, skrypt PowerShell nie może być niestety wywołany bezpośrednio przez Subversion. Do tego konieczny jest pośredni plik wsadowy (**batch file**). I tutaj również nie obyło się bez niespodzianek. Pierwsza wersja pliku prezentowała się tak:

```dos
        powershell -noprofile ..\..\scripts\pre-commit.ps1 %1 %2
        exit errorlevel
```

Okazało się jednak, iż ścieżka dostępu do katalogu ze skryptami nie jest tworzona poczynając od miejsca wywołania pliku wsadowego, tylko od ścieżki **C:\Windows\system32**. W ramach plików wsadowych można skorzystać ze zmiennej **%CD%**, która zawiera aktualny katalog. W tym wypadku nic to jednak nie dało. Nie pomogło nawet zapisanie wartości **%CD%** w zmiennej tymczasowej skryptu i późniejsze wykorzystanie jej przy tworzeniu ścieżki. Rozwiązaniem okazało się natomiast skorzystanie z makra **%~dp0**. Ostatecznie zawartość pliku wsadowego ustabilizowała się na poniższym zapisie:

```dos
        powershell -noprofile  %~dp0\..\..\scripts\pre-commit.ps1 %1 %2
        exit errorlevel
```

Co ciekawe zupełnie nie sprawdziło się również polecane również sprawdzanie zmiennej `$LASTEXITCODE` zamiast **ERRORLEVEL**.

Wiemy już, iż przedstawiane rozwiązanie posiada jedną niedogodność – konieczne jest posiłkowanie się plikami wsadowymi aby osiągnąć zamierzony efekt. Czy można to pominąć? Ano można. Należy skorzystać z parametru `–Command` w czasie wywołania powłoki PowerShell w pliku wsadowym i zapisać wewnątrz ciągu "& " cały kod skryptu pamiętając, aby wszystko znalazło się w jednej linii oraz, by kolejne bloki kodu oddzielone były znakiem średnika.

```dos
        powershell.exe -noprofile -command "& {$message = svnlook.exe log -t $args[1] $args[0];if($message -notmatch '[a-zA-Z0-9]'){[System.Console]::Error.WriteLine('Brak opisu poczynionych zmian.');$host.SetShouldExit(1)}}" %1 %2
        exit errorlevel
```

Dla skrócenia zapisu parametry skryptu przekazuję bezpośrednio do aplikacji **svnlook.exe** bez tworzenia zmiennych pomocniczych `$repositoryPath` i `$transactionName`.

Podsumowanie? Cóż. Skrypty przechwytujące (**hook scripts**) w PowerShell to chyba jednak w tym przypadku sztuka dla sztuki. Szybciej i sprawniej będzie skorzystanie z [kodu C#]({{< ref "hook-scripts-w-csharp" >}}). Choć zawsze można się czegoś pożytecznego przy okazji nauczyć.