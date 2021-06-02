---
title: Określanie dnia tygodnia z pogardą dla DATEFIRST
date: 2008-01-17T11:47:00+01:00
languageCode: pl-pl
description: Poprawne ustalenie dnia tygodnia w skryptach MS SQL
categories:
  - Databases
tags:
  - SQL Server
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/01/17/okre-lanie-dnia-tygodnia-z-pogard-dla-datefirst.aspx
disqus_disable: true
---

Do szału doprowadza mnie określanie dnia tygodnia w MS SQL. Zmiany ustawienia domyślnego pierwszego dnia tygodnia z niedzieli na poniedziałek nie są ustawiane globalnie dla bazy (chyba, że o czymś nie wiem) i nie mogą być wywoływane np. w funkcjach użytkownikach. Jedne zapytania wywoływane są w Reporting Services, inne w Integration Services, inne jeszcze gdzie indziej. Szału można dostać. Stąd pomysł na poniższy kod, który zwraca dnie tygodnia numerując je od cyfry jeden:

```sql
    CREATE FUNCTION GetDayOfWeek
    (
        @Value datetime
    )
    RETURNS tinyint
    AS
    BEGIN
        RETURN DATEDIFF(day, '1753-01-01', @Value) % 7 + 1
    END
```

Funkcja `DATEDIFF` z parametrem `day` zwraca liczbę dni pomiędzy minimalną dopuszczalną datą dla typu `datetime` a datą podaną parametrem `@Value`. Dzielenie modulo określa, który dzień tygodnia. Musimy dodać wartość jeden, ponieważ 1 stycznia 1753 roku to poniedziałek, w związku z czym różnica pomiędzy poniedziałkiem a poniedziałkiem z `@Value` jest zero, a my dla poniedziałku oczekujemy liczby jeden.