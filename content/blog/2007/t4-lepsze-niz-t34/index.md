---
title: T4 lepsze niż T34
date: 2007-04-23T10:20:00+02:00
lastmod: 2018-09-17T14:41:00+02:00
languageCode: pl-pl
description: Wykorzystanie szablonów tekstowych T4 Text Template Transformation Toolkit do generowania kodu warstwy danych w .NET C# Visual Studio
categories:
  - Development
tags:
  - .NET
  - Generowanie kodu
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/04/23/t4-lepsze-ni-t34.aspx
disqus_disable: true
---

Poniższy tekst bazuje na artykule mojego autorstwa z [drugiego numeru](http://zine.net.pl/files/folders/zine2007/entry62.aspx) gazetki [zine.net](http://zine.net.pl). Względem oryginału zrezygnowano jedynie z odwoływanie się do kodu źródłowego jako do rysunku oraz zmieniono numerację wstawionych zrzutów ekranu.

Wszelkie nazwy własne oraz podane wersje aplikacji i bibliotek odzwierciedlają stan na koniec 2006 roku.

## Wstęp

T4 oznacza Text Template Transformation Toolkit. Jest to dostarczany przez Microsoft w postaci biblioteki Microsoft.VisualStudio.TextTemplating.dll silnik do generowania tekstu na podstawie zdefiniowanych przez użytkownika szablonów. Szablon jest plikiem, który zawiera dyrektywy, wykonywany przez interpreter kod oraz bloki tekstu.

Microsoft udostępnia szablony tekstowe w ramach Visual Studio 2005 SDK (konkretnie jako część Domain-Specific Language (DSL) Tools) oraz w Guidance Automation Toolkit. Rozszerzenia te przeznaczone są głównie do środowiska programistycznego jakim jest Visual Studio 2005. Standardowe miejsce instalacji silnika T4 to %Program Files%\Common Files\Microsoft Shared\TextTemplating\1.1.

W artykule zostanie pokazane w jaki sposób zintegrować z rozwiązaniem (solution) tworzonym w Visual Studio 2005 projekt umożliwiający generowanie dowolnych plików wyjściowych na podstawie zdefiniowanego modelu oraz utworzonych szablonów.

## Problem

Załóżmy, iż budujemy rozwiązanie, które będzie składało się z bazy danych, klas rezydujących w ramach logiki biznesowej a reprezentujących tabele z bazy danych oraz warstwy dostępu do danych, która korzystając między innymi z poleceń SQL posiadać będzie możliwości pobierania i składowania danych w warstwie danych.

Jeden z wielu sposobów podejścia do tego zagadnienia polegałby mniej więcej na tym, iż dla jednej tabeli wykonywalibyśmy następujące kroki (dla uproszczenia przyjęto, iż jednej tabeli odpowiada tylko jedna klasa odwzorowująca):

1. Utworzenie skryptu SQL tworzącego tabelę;
1. Utworzenie testu klasy reprezentującej wiersz tabeli;
1. Utworzenie klasy reprezentującej wiersz tabeli;
1. Utworzenie testu klasy odpowiedzialnej za komunikację z bazą danych;
1. Utworzenie klasy odpowiedzialnej za komunikację z bazą danych.

Modyfikacja jednego pola w bazie danych oznacza dla nas konieczność wprowadzenia zmian we wszystkich wymienionych powyżej klasach – musimy przy tym zawsze pamiętać, które pliki tego wymagają. Do tego dochodzi możliwość popełnienia błędu w czasie modyfikacji skryptów SQL, które generalnie nie są sprawdzane pod kątem poprawności przez środowisko programistyczne. Nie bez znaczenia jest również fakt, iż znaczna część pracy przy budowaniu takiego szkieletu aplikacji polega na przekopiowywaniu dużych partii kodu z definicji jednej klasy do drugiej. Jednym słowem nawet najmniejsza zmiana powoduje za sobą zbyt dużą liczbę koniecznych do wykonania czynności, w czasie których możemy popełnić błąd.

Rozwiązaniem, które może pomóc nam uporać się z problemami powtarzalności, monotonii i błędów w życiu programisty są wspomniane na początku artykułu szablony tekstowe. W ramach interesującego nas rozwiązania tworzymy projekt, który umożliwi nam generowanie kodu na żądanie.

## Nowy projekt

Przygotowanie do automatycznego generowania plików oznacza wykonanie wpierw następujących czynności:

* Zdefiniowanie struktury, która będzie w sposób wystarczający opisywała model;
* Przygotowanie szablonów, na podstawie których następnie zostaną wygenerowane odpowiednie pliki wyjściowe;
* Uruchomienie generatora kodu.

## Definicja struktury

Pierwszym krokiem będzie zdefiniowanie struktury. Nasz model (`definition`) będzie składał się z dowolnej liczby encji (`entity`) zawierających pola (`field`). Dane te w procesie transformacji zostaną przekształcone na skrypt bazy danych oraz odpowiednie klasy warstwy biznesowej i danych. Naturalnym formatem dla definicji struktury, ze względu na możliwości budowania drzewiastych powiązań, są pliki w formacie XML. Rysunek 1 przedstawia niewielki fragment takiego opisu.

{{< figure src="images/t4_rysunek1_m.png" title="Fragment definicji struktury w formacie xml z podpowiadaniem" >}}

Jak widzimy środowisko Visual Studio może podpowiadać nam dostępne w danej chwili znaczniki. Dzięki temu unikamy problemów związanych z zapomnieniem nazwy czy pomyłką w pisowni. Aby osiągnąć ów efekt należy do projektu dodać plik typu **XML Schema**, w którym zdefiniujemy, które elementy i atrybuty definicji struktury są dozwolone, jakie wartości mogą przyjmować etc.

{{< figure src="images/t4_rysunek2_m.png" title="Schemat xsd struktury" >}}

Następnie we właściwościach (zakładka **Properties**) pliku XML umieszczamy odwołanie do utworzonego schematu (**Schemas**) dzięki czemu umożliwiamy edytorowi VS2005 sprawdzanie w czasie edycji definicji poprawności wpisywanych danych, podpowiadanie elementów, atrybutów oraz dostępnych wartości.

Jeśli planujemy tworzony generator wykorzystywać również w innych rozwiązaniach, warto plik XSD schematu włączyć do pliku wykonywalnego jako zasób (polecenie **Properties** menu podręcznego projektu, następnie zakładka **Resources**).

## Wczytanie struktury

Mając zdefiniowaną strukturę następnym krokiem jest wczytanie jej z dysku do pamięci. Korzystamy przy tym z możliwości dostarczanych przez przestrzeń nazw `System.Xml.Serialization`, która zawiera klasy umożliwiające serializację obiektów do formatu XML. Tworzymy w związku z tym dla każdego elementu z pliku XML klasę, której właściwościami są atrybuty i elementy tegoż elementu.

Poniżej mamy przykład klasy, która reprezentuje element `definition`. Powiązanie pomiędzy atrybutami XML a polami klasy zapewnia atrybut `XmlAttribute` (`XmlAttributeAttrubite`). Odwołanie do elementów uzyskujemy korzystając z atrybutu `XmlElement` (`XmlElementAttribute`).

```csharp
    [XmlRoot("definition", IsNullable = false, Namespace = "DefinitionSchema.xsd")]
    public class Definition
    {
        /// <summary>
        /// Lista encji.
        /// </summary>
        [XmlElement("entity")]
        public List<Entity> Entities;

        /// <summary>
        /// Opis struktury.
        /// </summary>
        [XmlAttribute("description")]
        public string Description;

        /// <summary>
        /// Przestrzeń nazw, z którą będą generowane pliki źródłowe z kodami.
        /// </summary>
        [XmlAttribute("namespace")]
        public string Namespace;
    }
```

Posiadając już utworzone klasy `Definition`, `Entity`, `Field` można wczytać je do pamięci. Operację tę wykonuje następujący kod:

```csharp
        public static Definition ReadDefinition()
        {
            XmlReaderSettings settings = new XmlReaderSettings();
            settings.ValidationType = ValidationType.Schema;
            // settings.Schemas.Add(null, @"Schema\DefinitionSchema.xsd");
            settings.Schemas.Add(null, XmlReader.Create(new StringReader(
                Properties.Resources.DefinitionSchema)));
            XmlReader reader = XmlReader.Create("Definition.xml", settings);
            XmlSerializer serializer = new XmlSerializer(typeof(Definition));
            return (Definition)serializer.Deserialize(reader);
           }
```

Jak widzimy poza samym wczytaniem następuje również sprawdzenie poprawności pliku zawierającego definicję struktury, przy czym linia piąta zawiera kod dla przypadku, w którym plik schematu odczytujemy z dysku, zaś linie szósta i siódma korzystają z wariantu kiedy schemat stanowi zasób pliku wykonywalnego.

Można również opisany powyżej proces odwrócić. Wpierw tworzymy w pliku XML dokładną definicję struktury. Następnie wybieramy w menu polecenie **XML** a następnie **Create Schema**. Utworzony plik zapisujemy na dysku i dołączamy do projektu. Na koniec korzystając z narzędzia **XML Schema Definition Tool** (xsd.exe) generujemy klasy na podstawie zdefiniowanego pliku schematu i wczytujemy dane do pamięci.

## Szablony

Generowany kod powstaje w procesie transformacji na podstawie i przy współudziale szablonów tekstowych. Szablony poza tekstem zawierają dyrektywy (znaczniki <#@...#>), które przede wszystkim ustawiają parametry transformacji, importują konieczne przestrzenie nazw etc. Korzystając ze znaczników <#…#>, <#=…#>, <#+…#> możemy do szablonu wstawić kod język C# lub Visual Basic, który w czasie transformacji może wpływać na tworzony wyjściowy strumień znakowy. T4 umożliwia nam również zdefiniowanie własnego analizatora dyrektyw.

Kod poniżej pokazuje początek szablonu **Entity.cs.t4**, który generuje klasy odwzorowujące zdefiniowane encje.

```csharp
    <#@ Assembly Name="t4.exe" #>
    <#@ import namespace="T4" #>
    <#@ import namespace="T4.Schema" #>
    <#@ import namespace="T4.Helpers" #>
    <#
        Definition definition = DefinitionLoader.ReadDefinition();
    #>
    <#@ include file="Templates\Copyright.txt" #>
    using System;
    using System.Collections.Generic;
    using System.Text;

    <#@ include file="Templates\Warning.txt" #>

    namespace <#= definition.Namespace #>.Business.Entities
    {
    <#
    // Generujemy tyle klas ile mamy zdefiniowanych encji.
    foreach(Entity entity in definition.Entities)
    {
    #>

        /// <summary>
        /// <#= entity.Description #>
        /// </summary>
        partial class <#= entity.Name #>
        {
```

Pierwsze cztery linie informują silnik T4 gdzie należy szukać w procesie transformacji wywoływanych metod pomocniczych. Następnie w linii szóstej ładujemy do pamięci zdefiniowaną strukturę. W linii ósmej oraz trzynastej włączamy do szablony zawartość zdefiniowaną w plikach zewnętrznych. Dalej następuje w pętli (linia dziewiętnasta) generowanie klas.

## Generowanie kodu

Na koniec pozostało nam wywołanie silnika T4. W Visual Studio 2005 SDK jak i w Guidance Automation Toolkit proces ten przebiega identycznie.

```csharp
                Engine engine = new Engine();
                Host host = new Host();

                ProcessTemplate(engine, host, @"Entity.cs.t4");
```

Wpierw tworzymy wpierw silnik T4 (`Engine`) oraz host czyli klasę implementującą interfejs `ITextTemplatingEngineHost` a odpowiadającą między innymi za interpretację dyrektyw znajdujących się w szablonach. Następnie wywołujemy metodę `ProcessTemplate` silnika przekazując mu zawartość szablonu oraz host.

```csharp
            private static void ProcessTemplate(Engine engine, Host host, string fileName)
            {
                string filePath = string.Format(@".\Templates\{0}", fileName);

                host.TemplateFileName = filePath;
                string input = File.ReadAllText(filePath, Encoding.Default);
                string ouput = engine.ProcessTemplate(input, host);
                File.WriteAllText(Path.GetFileNameWithoutExtension(fileName), ouput, Encoding.Default);
            }
```

W przypadku Visual Studio 2005 SDK klasę implementującą interfejs `ITextTemplatingEngineHost` musimy utworzyć samodzielnie. Nie jest to na szczęście trudne tym bardziej, iż w pomocy znajdziemy w pełni funkcjonalny przykład. Zwrócić należy przy tym uwagę na metodę `ProvideTemplatingAppDomain`, która dla każdego wywołania (dla każdego szablonu) tworzy standardowo nową domenę aplikacji. Jeśli chcemy, aby szablony miały dostęp do danych przygotowanych np. przez metodę wywołującą transformację należy `AppDomain.CreateDomain` zamienić kodem `AppDomain.CurrentDomain`.

Jeśli mamy zainstalowany GAT możemy skorzystać z dobrodziejstw biblioteki Microsoft.Practices.RecipeFramework.VisualStudio.Library.dll a konkretnie z przestrzeni nazw `Microsoft.Practices.RecipeFramework.VisualStudio.Library.Templates`. Znajdziemy tam bowiem implementację interfejsu `ITextTemplatingEngineHost` w postaci klasy `TemplateHost`, która dodatkowo używa wewnętrznie klasy `PropertiesDirectiveProcessor` dziedziczącej z `DirectiveProcessor` a umożliwiającej przekazywanie do szablonu dodatkowych parametrów.

## Podsumowanie

Opisywane w tym artykule rozwiązanie jest jak najbardziej prawdziwe i wyewoluowało z własnego generatora kodu napisanego całkowicie na potrzeby tegoż jednego rozwiązania. Zastosowanie rozszerzeń do środowiska programistycznego Visual Studio 2005 zamiast rozwiązań własnych umożliwiło skupienie się nad samych projektem. Inną zaletą korzystania z takiego rozwiązania jest łatwiejsze wdrożenie nowego programisty w projekt.

Adresy i dodatkowe informacje:

* `http://msdn.microsoft.com/vstudio/extend` - rozszerzenia Visual Studio 2005;
* `http://msdn.microsoft.com/vstudio/DSLTools` - informacje dotyczące Domain Specific Language Tools;
* `http://msdn.microsoft.com/vstudio/teamsystem/workshop/gat/default.aspx` - strona poświęcona Guidance Automation Extensions oraz Guidance Automation Toolkit;
* `http://www.microsoft.com/downloads/details.aspx?FamilyID=DB996113-6E92-4894-9B7E-0DEBB614D72F&displaylang=en` - Web Services Software Factory. Przykładowe wykorzystanie między innymi możliwości T4.