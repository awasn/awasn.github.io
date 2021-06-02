---
title: Typy wyliczane czy klasy
date: 2008-10-24T12:20:00+02:00
lastmod: 2018-09-17T13:28:00+02:00
languageCode: pl-pl
description: Definiowanie rozbudowanych stałych w programie przy wykorzystaniu typów wyliczanych i klas
categories:
  - Development
  - Events
tags:
  - .NET
  - Design patterns
  - Meetings
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/10/24/typy-wyliczane-czy-klasy.aspx
disqus_disable: true
---

Tak to już jest, iż programując bardzo często stajemy przed koniecznością wyboru rozwiązania, będąc  gdzieś w połowie drogi pomiędzy "najlepszymi technikami". Jeden z takich przypadków, ale bez wybrania najlepszej drogi, chciałbym opisać poniżej.

Załóżmy, iż budujemy aplikację służącą sprzedaży  Na początek będziemy wykorzystywać dwa typy dokumentów: paragon i fakturą.  W kodzie tworzymy odpowiadający temu typ wyliczany:

```csharp
    internal enum DocumentType
    {
        Receipt,
        Invoice
    }
```

Następnie pojawiają się nowe wymagania. Potrzebujemy powiązać z typami dokumentów skrót, czyli kilkuznakowy identyfikator, który będzie wykorzystywany:

* W czasie budowania numeru dokumentu;
* Być może w czasie wyświetlania informacji o dokumentach;
* Być może w czasie konwersji danych do/z systemów zewnętrznych;

oraz nazwę dokumentu, która potrzebna będzie przy:

* Być może drukowaniu dokumentów;
* Być może wyświetlaniu informacji o dokumentach.

Jak widzimy wymagania te wychodzą z różnych części programu. Słowa "być może" są użyte powyżej ponieważ to klient (użytkownik oprogramowania) będzie decydował (przy zakupie?) co i gdzie ma mu się wyświetlać.

Pierwsze rozwiązanie polega na zastosowaniu specjalnej klasy pomocniczej,  która przyjmując w wywołaniach metod typ dokumentu zwracałaby właściwe wartości. Prosty przykład użycia:

```csharp
            DocumentType documentType = DocumentType.Invoice;
            Console.WriteLine(DocumentTypeHelper.GetCode(documentType));
            Console.WriteLine(DocumentTypeHelper.GetName(documentType));
```

Oczywiścia klasa pomocnicza nie musi być statyczna. Do dyspozycji mamy również nowości języka C# w wersji trzeciej czyli metody rozszerzające (extension methods):

```csharp
            Console.WriteLine(documentType.GetCode());
            Console.WriteLine(documentType.GetName());
```

Możemy również zastosować refaktoryzację naszego typu wyliczanego do klasy. Rozwiązanie to jest bardzo popularne zwłaszcza w językach, które nie umożliwiają definiowania dozwolonych zakresów wartości dla zmiennych i parametrów metod.

```csharp
    internal class DocumentType
    {
        public static readonly DocumentType Receipt =
            new DocumentType("PA", "Paragon");

        public static readonly DocumentType Invoice =
            new DocumentType("FK", "Faktura VAT");

        public string Code { get; private set; }
        public string Name { get; private set; }

        public DocumentType(string code, string name)
        {
            Code = code;
            Name = name;
        }
    }
```

Jako ciekawostkę można w tym miejscu powiedzieć, iż kompilator C# zamienia typy enumerowane na poniższe rozwiązanie  (mniej więcej):

```csharp
    internal struct DocumentType : System.Enum
    {
        public const DocumentType Receipt = (DocumentType) 0;
        public const DocumentType Invoice = (DocumentType) 1;
    }
```

Oczywiście powyższy kod (czy też raczej pseudokod) się nie skompiluje (nie można dziedziczyć po `System.Enum`), ale tak to wewnątrz CLR wygląda.

Wracając do naszej aplikacji. A co począć jeśli pojawiają nam się dodatkowe wymagania w projekcie dotyczące typów dokumentów:

* Co w przypadku jeśli użytkownik aplikacji zażyczy sobie aby zamiast skrótu FK był kod FS;
* Co jeśli chcemy dodać nowy typ dokumentu;
* Co jeśli musimy dodać nową właściwość opisującą typ dokumentu.

W którą stronę podążać i jakie zmiany w kodzie przeprowadzić? Pytania te wydały mi się na tyle ciekawe, iż postanowiłem na 32 spotkaniu [Warszawskiej Grupy .NET](http://www.wg.net.pl) spróbować przeprowadzić małą prezentację problemu wraz dyskusją na ten temat. Całość trwała około 20 minut i oto proponowane przez uczestników dyskusji rozwiązania:

1. Powrót do typów wyliczanych i klas pomocniczych ponieważ umieszczanie właściwości opisujących dany typ w klasie jako pól statycznych narusza zasadę pojedynczej odpowiedzialności;
2. Pobieranie wartości łańcuchowych w momencie tworzenia instancji klas z zasobów zewnętrznych. Tudzież tworzenie klas dla danych typów w konstruktorze statycznym, który wiedziałby (zasoby zewnętrzne) z jakimi wartościami je kreować.
3. Rezygnacja ze statycznych pól tylko do odczytu. Przeniesienie typu dokumentu do klasy:

    ```csharp
        internal class DocumentTypeValue
        {
            public DocumentType DocumentType { get; private set; }
            public string Code { get; private set; }
            public string Name { get; private set; }

            public DocumentTypeValue(DocumentType documentType,
                string code, string name)
            {
                DocumentType = documentType;
                Code = code;
                Name = name;
            }
        }
    ```

4. Opisanie elementów typu wyliczanego utworzonymi atrybutami:

    ```csharp
        internal enum DocumentType
        {
            [Code("PA")]
            [Name("Paragon")]
            Receipt,

            [Code("FK")]
            [Name("Faktura VAT")]
            Invoice
        }
    ```

Moje subiektywne odczucia z dyskusji są takie, iż minimalnie wygrały rozwiązania numer 2 i 3. A co Wy o tym sądzicie?