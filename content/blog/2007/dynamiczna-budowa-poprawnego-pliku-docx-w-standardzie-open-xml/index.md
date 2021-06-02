---
title: Dynamiczna budowa poprawnego pliku docx w standardzie Open XML
date: 2007-11-24T00:22:00+01:00
languageCode: pl-pl
description: Formularze i programowa dynamiczna budowa poprawnego pliku docx w standardzie Open XML
categories:
  - Development
  - Tools
tags:
  - .NET
  - Open XML
  - MS Office
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/11/24/dynamiczna-budowa-poprawnego-pliku-docx-w-standardzie-open-xml.aspx
disqus_disable: true
---

Konieczność utworzenie dokumentu, który byłby stały w treści a gdzie tylko niektóre elementy, np. imię i nazwisko, powinny się zmieniać dynamicznie jest sytuacją nieobcą chyba każdemu programiście czy informatykowi. Rozwiązań takiego problemu jest wiele.

Ostatnio w poszukiwaniu absolutu zainteresowałem się pakietem Microsoft Office, a konkretnie aplikacją Word 2007. Cały pakiet do zapisywania danych wykorzystuje nowy format plików bazujący na standardzie [Open XML](http://www.ecma-international.org/publications/standards/Ecma-376.htm). Standardzie nota bene promowanym przez sam Microsoft. Nowe pliki łatwo poznać po literze x dodawanej do rozszerzenia znanego z wcześniejszych wersji. W przypadku Worda będzie to więc docx.

Wewnętrzna zawartość plików w formacie Open XML będzie dla wielu zaskoczeniem. Jest to bowiem ściśle określona struktura katalogów i plików, głównie w formacie xml, potraktowana pakietem kompresującym zip. Pliki te zawierają konfigurację, style oraz treść.

{{< figure src="images/OpenXML_1.png" title="Zawartość nowego dokumentu docx" >}}

Zdziwieni? Proponuję wziąć dowolny plik docx, zmienić jego rozszerzenie na zip a następnie obejrzeć zawartość. To co wpiszemy w edytorze zawarte jest zaś w pliku **word/document.xml**.

Zalety nowego formatu plików to między oddzielenie prezentacji od danych. Brzmi to oczywiście bardzo znajomo. Ponieważ struktura pliku docx jest jasno określona można pokusić się o ręczne vel programowe stworzenie poprawnego dokumentu w formacie Open XML. Co prawda definicja standardu liczona jest w setkach stron, ale wymaganych elementów nie jest dużo. Na stronach [msdn](https://docs.microsoft.com) można znaleźć między innymi artykuł [Walkthrough: Word 2007 XML Format](https://docs.microsoft.com/en-us/previous-versions/office/office-12//ms771890(v=office.12)), który poza opisem formatu Open XML pokazuje jak utworzyć poprawny plik docx. Jednym słowem jesteśmy wstanie dynamicznie tworzyć pliki programu Word 2007.

Pójdźmy krok dalej. Okazuje się, iż możemy wewnątrz struktury docx pomiędzy plikiem zawierającym treść naszego dokumentu utworzyć łącznik do pliku o dowolnej, poprawnej strukturze w formacie xml, który będzie zawierał zmienne, z naszego punktu widzenia, dane. Trochę to brzmi enigmatycznie, ale już tłumaczę o co chodzi. Mianowicie możemy w dokumencie Word zdefiniować formularz, którego pola mogą być powiązane poprzez XPath z danymi umieszczonymi w pliku xml. I co ciekawe możemy plik xml zawierający dane dla formularza programowo zmieniać! Zobaczmy więc jak to osiągnąć.

## Szablon dokumentu

Po pierwsze utwórzmy nowy dokument w programie Microsoft Word i nazwijmy go **person_template.docx**. Zanim rozpoczniemy pracę musimy jeszcze uaktywnić w programie Word zakładkę **Deweloper** (jakby nie można tego nazwać Programista). Klikamy **Przycisk pakietu Office** i wybieramy polecenie **Opcje programu Word**.

{{< figure src="images/OpenXML_WordOption.png" title="Opcje programu Word - zakładka Deweloper" >}}

Następnie zaznaczamy opcję **Pokaż kartę Deweloper na Wstążce**. Dzięki temu w programie Word pojawi nam się dodatkowa zakładka.

Mając już skonfigurowany edytor wpiszmy do dokumentu zwrot **Szanowny Pan**. Ze spacją na końcu. Tego typu sformułowania widnieją bardzo często na różnego rodzaju korespondencji. Teraz należy poinformować Word, iż chcemy utworzyć dwa dynamiczne pola tekstowe. Przełączamy się na zakładkę **Deweloper** i wybieramy kontrolkę **Tekst**.

{{< figure src="images/OpenXML_TextControl.png" title="Kontrolka tekstowa formularza dokumentu docx" >}}

W miejscu kursora program wstawi odpowiedni znacznik. Niech tekst, który będzie wyświetlany na ułatwienia tworzenia dokumentu dla uproszczenia będzie brzmiał **imię** (po powiązaniu z plikiem xml ten napis zostanie automatycznie usunięty). Aby ułatwić sobie pracę możemy jeszcze włączyć **Tryb projektowania** dzięki czemu wokół kontrolek pojawią znaczniki. Mając wstawioną kontrolkę klikamy ją  prawym klawiszem myszki i wybieramy polecenie **Właściwości** (polecenie to możemy również wybrać z zakładki **Deweloper**).

{{< figure src="images/OpneXML_ControlProperty.png" title="Właściwości kontrolki formularza docx" >}}

Wpisujemy tytuł **Imię**. Ten napis pojawi się jeśli wybierzemy znacznik. Etykieta **Tag** zawierać będzie zaś nazwę, która umożliwi nam połączenie z danymi. Dla imienia będzie to **firstName**. Następnie wstawiamy kontrolkę tekst, aby dodać pole identyfikujące nazwisko (etykieta **Tag** nazwana **lastName**). Nasz formularz po tych operacjach powinien wyglądać tak jak na obrazku poniżej:

{{< figure src="images/OpenXML_Template.png" title="Szablon przykładowego dokumentu docx z formularzem" >}}

Proponuję również obejrzeć zawartość powstałego dokumentu. Zobaczy, iż względem wersji zawierającej tylko tekst zasadniczo się on zmienił. Ale najciekawsze dopiero przed nami.

{{< figure src="images/OpenXML_2.png" title="Dokument docx z formularzem" >}}

Teraz bowiem przyszedł czas na to, aby powiązać dane na formularzu z elementami struktury xml. Wpierw zbudujmy prosty plik w formacie xml. Jak widzimy nazwy elementów odpowiadają nazwom tagów dokumentu Word. Zgodność ta nie jest konieczna, ale zasadniczo ułatwi nam to pracę, jeśli tagów byśmy mieli w dziesiątkach czy setkach. Plik będzie pusty, bez danych i będzie się nazywał **person_data.xml**:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <root>
        <person>
            <firstName></firstName>
            <lastName></lastName>
        </person>
    </root>
```

## Powiązanie formularza z danymi

Następny krok będzie od nas wymagał pobrania ze stron <https://archive.codeplex.com/> pakietu [Word 2007 Content Control Toolkit](https://archive.codeplex.com/?p=dbe), który połączy (zmapuje) nam wstawione elementy formularza (kontrolki) z konkretnymi elementami powyższego pliku xml. Po instalacji i uruchomieniu programu Word 2007 Content Control Toolkit otwieramy nasz plik w formacie docx. Lewa strona aplikacji pokaże nam wszystkie znalezione elementy formularza - kontrolki. Widzimy nazwy tagów, które nadaliśmy elementom formularza oraz pustą kolumnę **XPath** co oznacza brak powiązań z danymi.

{{< figure src="images/OpenXML_ContentControls.png" title="Formularz docx - nie skojarzony z danymi" >}}

Teraz musimy dodać utworzoną strukturę xml. W tym celu w prawej części aplikacji wybieramy polecenie **Create a new Customer XML Part**. Następnie w zakładce **Edit View** ładujemy strukturę xml do programu. Dalej mapujemy gałęzie xml do kontrolek. W tym celu wpierw wybieramy kontrolkę, a następnie używając drag & drop przenosimy wybrany element struktury na interesującą nas kontrolkę.

{{< figure src="images/OpenXML_XPath.png" title="Formularz docx - skojarzony z danymi poprzez XPath" >}}

Na koniec zapisujemy wprowadzone zmiany. Jeśli teraz obejrzelibyśmy zawartość pliku docx to zobaczymy, iż w strukturze katalogów pojawił się folder **customerXml**, który posiada plik **item1.xml** o zawartości zgodnej z naszym dodawanym plikiem xml. Oczywiście nic nie stoi na przeszkodzie, aby samemu ów formularz wypełnić z poziomu programu Word. Nie powinno być niespodzianką, iż wówczas plik **item1.xml** będzie uzupełniony o wartości, które wpisaliśmy.

{{< figure src="images/OpenXML_3.png" title="Dokument docx z formularzem i danymi" >}}

## Programowa zmiana danych

Przyszedł czas na ostatni etap, czyli na dynamiczną modyfikację szablonu pliku docx z uwzględnieniem danych dostarczanych z zewnętrznych źródeł. Teraz posłużymy się narzędziem Visual Studio. Wersja 3.0 platformy .NET dostarcza komponent **WindowsBase.dll**, który poprzez przestrzeń nazw `System.IO.Packaging` umożliwia manipulowanie zawartością plików w formacie Open XML. Wszystko co musimy zrobić to dodać referencję do tej biblioteki z naszej aplikacji. A oto przykładowe źródła:

```csharp
    Package package = Package.Open("person_template.docx");
    try
    {
        Uri uri = new Uri("/customXML/item1.xml", UriKind.Relative);
        PackagePart xmlPart = package.GetPart(uri);
        Stream stream = xmlPart.GetStream();
        stream.SetLength(0);
        XmlDocument document = new XmlDocument();
        document.Load("person_data.xml");
        document.Save(stream);
        stream.Close();
    }
    finally
    {
        package.Close();
    }
```

Powyższy kod tworzy obiekt reprezentujący pakiet jakim jest dokument docx i  ów dokument otwiera. Następnie wyszukuje w pakiecie plik zawierający powiązane z formularzem dane i zwraca go naszej aplikacji jako strumień. Na koniec wczytuje plik xml z nowymi danymi i uaktualnia pakiet (tak naprawdę uaktualnia odczytany wcześniej strumień).

Jeśli wczytana struktura będzie miała poniższą zawartość:

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <root>
        <person>
            <firstName>Arkadiusz</firstName>
            <lastName>Waśniewski</lastName>
        </person>
    </root>
```

to po wykonaniu kodu i otworzeniu w programie Word dokumentu docx zobaczymy, iż zawartość elementów `firstName` i `lastName` pojawi się jako dopełnienie pierwotnego napisu.

## Podsumowanie

Równie ciekawie, jeśli nie ciekawiej, prezentują się opisane powyżej możliwości jeśli mamy przygotowane dokumenty, które muszą być wypełniane przez inne osoby. Wyobraźmy sobie formularz uzupełniany przed przyjęciem do szpitala. Wpisane w określonych polach dane automatycznie lądują w pliku xml, dzięki czemu bez parsowania czy przepisywania mamy dostęp do informacji, które mogą być bardzo szybko przetworzone.

Mam nadzieję, iż przedstawione powyżej rozwiązanie zachęci do przyjrzenia się bliżej możliwościom formatu Open XML.

Adresy warte odwiedzenia na początek:

* <http://openxmldeveloper.org/>;
* `http://msdn.microsoft.com/msdnmag/issues/07/02/OfficeSpace/default.aspx` - Building Office Open XML Files;
* `http://msdn2.microsoft.com/en-us/office/aa905545.aspx` - XML in Office Developer Portal;
* <https://docs.microsoft.com/en-us/previous-versions/office/developer/office-2007/aa338205(v=office.12)> - Introducing the Office (2007) Open XML File Formats;
* `http://www.devx.com/MicrosoftISV/Article/30907/2046` - 5 Cool Things You Must Know About the New Office 2007 File Formats.