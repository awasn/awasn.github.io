---
title: ReSharper ostrzega - Possible NullReferenceException
date: 2008-01-09T23:15:00+01:00
languageCode: pl-pl
description: ReSharper pomaga uniknąć błędów w kodzie dodając ostrzeżenia
categories:
  - Development
  - Tools
tags:
  - .NET
  - ReSharper
  - Software testing
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/01/09/resharper-ostrzega-possible-nullreferenceexception.aspx
disqus_disable: true
---

Kontekst rozważań jest następujący. Metoda (wersja uproszczona poniżej) buduje ściśle określoną sekwencję sterującą. Dokumentacja mówi, iż sekwencja ta musi zawierać dokładnie osiem parametrów oddzielonych średnikiem. Dopuszcza się przy tym parametry puste. Użytkownik nie jest karany (np. wyjątek) za brak parametrów. Nie spotka go też przykra niespodzianka, jeśli poda zbyt dużo parametrów.

```csharp
    public void SampleReport(string[] names)
    {
        const byte maxNumberOfNames = 8;

        int numberOfNames = names == null ? 0 : names.Length;

        byte[] sequence;

        using (SequenceBuilder builder = new SequenceBuilder())
        {
            for (int i = 0; i < maxNumberOfNames; i++)
            {
                if (i < numberOfNames)
                {
                    builder.AppendParameter(names[i]);
                }
                builder.AppendSemicolon();
            }

            sequence = builder.BuildUp();
        }

        printController.Print(sequence);
    }
```

Pewnym zdziwieniem w związku z tym było dla mnie zgłoszenie przez ReSharper ostrzeżenia **Possible 'System.NullReferenceException'** dla linii numer 15. Wśród sugerowanych akcji zaś pojawiła się propozycja sprawdzenia czy parametr `names` nie jest `null`. W pierwszej chwili podpowiedź ta zaskoczyła mnie. Kod jest poprawny. Po dłuższym zastanowieniu stwierdziłem, iż ostrzeżenie programu ReSharper ma sens. To, że parametr `names` w tym miejscu programu nigdy nie będzie miał wartości `null` jest w teraz oczywiste. Za kilka miesięcy zaś może już tak nie być. Zamiast analizować kod można przecież wprost powiedzieć, iż w tym miejscu nie ma prawa nie być wartości. Jak to osiągnąć? Oczywiście poprzez zastosowanie asercji, które powinny być wykorzystywane do informowania siebie samego i innych programistów pracujących nad kodem, iż w danym konkretnym miejscu oczekujemy takich a nie innych wartości. Po wstawieniu właściwego sprawdzenia (linia numer 15) ReSharper usunął wcześniejsze zastrzeżenia.

```csharp
    public void SampleReport(string[] names)
    {
        const byte maxNumberOfNames = 8;

        int numberOfNames = names == null ? 0 : names.Length;

        byte[] sequence;

        using (SequenceBuilder builder = new SequenceBuilder())
        {
            for (int i = 0; i < maxNumberOfNames; i++)
            {
                if (i < numberOfNames)
                {
                    Debug.Assert(names != null);
                    builder.AppendParameter(names[i]);
                }
                builder.AppendSemicolon();
            }

            sequence = builder.BuildUp();
        }

        printController.Print(sequence);
    }
```

Asercja sprawdzana jest tylko w trybie **Debug**. W trybie **Release** pojawianie się takich okien jest w moim mniemaniu niedopuszczalne.