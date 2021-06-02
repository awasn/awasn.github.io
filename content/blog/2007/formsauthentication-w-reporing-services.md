---
title: FormsAuthentication w Reporing Services
date: 2007-11-15T14:41:00+02:00
languageCode: pl-pl
description: Przykład rozwiązania identyfikacji FormsAuthentication w Reporing Services
categories:
  - Databases
tags:
  - .NET
  - ASP.NET
  - Reporting Services
  - SQL Server
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/11/15/formsauthentication-w-reporing-services.aspx
disqus_disable: true
---

Domyślnie usługi raportujące Microsoft SQL Server 2005 obsługują jedynie identyfikację Windows. Na szczęście w [przykładach](http://www.codeplex.com/SqlServerSamples) dla Reporitng Services dostępnych na stronach <http://www.codeplex.com> dostępna jest implementacja rozszerzeń usług raportujących (interfejsy `IAuthenticationExtension` i `IAuthorizationExtension`), które umożliwiają skorzystanie z dobrodziejstw identyfikacji opartej o Forms. Jeśli zainstalujemy przykłady, to interesujące nas kody znajdziemy w katalogu **C:\Program Files\Microsoft SQL Server\90\Samples\Reporting Services\Extension Samples\FormsAuthentication Sample** zakładając oczywiście, iż wybraliśmy domyślną lokalizację.

Przykładowe rozwiązanie korzysta z własnej bazy danych do przechowywania informacji o użytkownikach. Zazwyczaj jednak jest tak, iż w identyfikacji Forms korzystamy z własnych danych. Wdrożenie rozszerzeń Reporting Services powoduje poważną ingerencję w strukturę i konfigurację usług raportujących serwera SQL. Zmiana bazy danych (nazewnictwo) oznacza zasadniczą modyfikację przykładowej aplikacji. A co w przyszłości? Jeśli zechcemy za jakiś czas dokonać kolejnych modyfikacji? Ciągłe zmiany w ramach Reporting Services nie są dobrym pomysłem.

Moja propozycja to zachowanie istniejącego przykładu w części weryfikującej użytkowników bez zmian. Modyfikacje natomiast wprowadzamy do skryptu tworzącego bazę danych. Usuwamy z niego kod tworzący tabelę `Users` oraz procedurę składowaną `RegisterUser` (wraz z uprawnieniami). Następnie modyfikujemy kod procedury składowanej `LookupUser`, która odpowiada za pobranie zaszyfrowanego hasła (hash) oraz wartości losowej (salt) utrudniającej złamanie hasła metodą słownikową. Zamiast sięgać do tabeli wewnątrz bazy rozszerzenia, pobierać będziemy dane z naszej własnej, zewnętrznej bazy danych. Dzięki temu wszelkie zmiany w przyszłości będą ograniczały się do niewielkiej modyfikacji kodu procedury `LookupUser` zamiast przebudowy kodu przykładowego rozszerzenia. Poniżej kod owej procedury przed modyfikacją:

```sql
CREATE PROCEDURE LookupUser
    @userName varchar(255)
AS
    SELECT PasswordHash, salt
    FROM Users
    WHERE UserName = @userName
GO
```

oraz po niej:

```sql
CREATE PROCEDURE LookupUser
    @userName varchar(255)
AS
    SELECT u.PasswordHash, u.PasswordSalt
    FROM OurDatabase.Users AS u
    WHERE u.Name = @userName
GO
```

PS. Dokumentacja opisująca wdrożenie rozszerzeń zawiera błąd w części dotyczącej modyfikacji pliku **Web.config** dla aplikacji Report Server. Jest podana niewłaściwa ścieżka dostępu do formularza logowania. Brakuje folderu **Pages**.