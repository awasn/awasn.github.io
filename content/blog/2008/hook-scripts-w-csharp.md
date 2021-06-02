---
title: Hook scripts w C#
date: 2008-02-12T16:48:00+01:00
languageCode: pl-pl
description: Skrypty przechwytujące hook scripts w .NET C# dla repozytoriów systemu kontroli wersji Subversion
categories:
  - Development
tags:
  - .NET
  - Subversion
series:
  - Subversion hook scripts
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/02/12/hook-scripts-w-c.aspx
disqus_disable: true
---

W strukturze katalogów składających się na repozytorium Subversion znajduje się folder o nazwie **hooks**. Zawiera on skrypty przechwytujące (**hook scripts**). Skrypty te wywoływane są w momencie zajścia w repozytorium określonego zdarzenia. Może to być np. żądanie zablokowania zasobu czy próba zatwierdzenia zmian. Przypisanie do zdarzenia następuje poprzez nadanie skryptowi odpowiedniej, rozpoznawalnej przez SVN nazwy. Mimo stosowania pojęcia skrypt, mogą to być tak naprawdę dowolne pliki wykonywalne. Nic nie stoi na przeszkodzie, abyśmy wykorzystali w tej roli język C#.

Domyślnie Subversion pozwala na zatwierdzenie zmian w repozytorium bez podania informacji (komunikatu). Najwłaściwszym miejscem do cofnięcia transakcji, która nie posiada wypełnionego logu jest zdarzenie **pre-commit** wywoływane przed ostatecznym zatwierdzeniem zmian. Tworzymy w związku z tym projekt typu **ConsoleApplication**, w którym zmieniamy wpierw nagłówek metody `Main` a następnie wypełniamy kodem:

```csharp
    static int Main(string[] args)
    {
        string repositoryPath = args[0];
        string transactionName = args[1];

        Process process = new Process();
        process.StartInfo.FileName = "svnlook.exe";
        process.StartInfo.Arguments = string.Format("log -t {0} {1}",
            transactionName, repositoryPath);
        process.StartInfo.UseShellExecute = false;
        process.StartInfo.RedirectStandardOutput = true;
        process.StartInfo.CreateNoWindow = true;
        process.Start();
        string output = process.StandardOutput.ReadToEnd();
        process.WaitForExit();

        Regex regex = new Regex("[a-zA-Z0-9]");
        if (regex.IsMatch(output))
        {
            return 0;
        }
        else
        {
            Console.Error.WriteLine("Brak opisu poczynionych zmian.");
            return 1;
        }
    }
```

Subversion dla tego zdarzenia w parametrach wywołania umieszcza informacje o ścieżce dostępu do repozytorium oraz o numerze przeprowadzanej transakcji. Wykorzystujemy te parametry do wywołania programy **svnlook**, który z parametrem **-t** przekaże nam strumień będący zawartością wpisanego przez użytkownika komunikatu. Jeśli nie ma żadnych informacji to zwracamy wartość różną od zera oraz wiadomość, którą chcemy wyświetlić zapominalskiemu użytkownikowi. Komunikat dla użytkownika musi być koniecznie przekazany na standardowe wyjście (strumień) błędów.

Aplikacje konsolowe są przez system uruchamiane każdorazowo w oknie. Aby tego uniknąć we właściwościach projektu, na zakładce **Application** zmieniamy **Output type** z **Console Application** na **Windows Application**.