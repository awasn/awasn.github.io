---
title: Metoda fabryki
date: 2007-12-27T21:27:00+01:00
lastmod: 2018-09-17T14:46:00+02:00
languageCode: pl-pl
description: Wzorzec metoda fabryki w .NET C#
categories:
  - Development
  - Events
tags:
  - .NET
  - Design patterns
  - Meetings
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/12/27/metoda-fabryki.aspx
disqus_disable: true
---

Poniższy tekst bazuje na prezentacji, którą przeprowadziłem 13 grudnia 2007 na połączonym XVI Spotkaniu [Warszawskiej Grupy .NET](http://groups.google.com/group/wg-net) + VII Spotkaniu [Polskiej Grupy Użytkowników SQL Server](http://plssug.org.pl/).

## Wstęp

Metoda fabryki (ang. Factory Method) jest, obok signletona czy budowniczego, wzorcem kreacyjnym odpowiadającym za tworzenie obiektów - instancji klas. Główne zadanie metody fabryki to oddzielenie procesu korzystania z obiektów od ich tworzenia.

Wyobraźmy sobie aplikację w wersji standardowej przeznaczoną do obsługi sprzedaży, w której dla kilku klientów dokonujemy modyfikacji. Każdy z nich wymaga własnej wersji klasy opisującej produkty. Ile miejsc w kodzie programu należałoby zmienić i jakie techniki zastosować (może kompilacja warunkowa)? Fakt zaś, iż dzisiejsze narzędzia programistyczne ułatwiają przeprowadzanie dużych modyfikacji źródeł nie zwalnia nas wcale od obowiązku dbania o czystość i przejrzystość kodu.

## Struktura

W celu skorzystania z wzorca Factory Method definiujemy interfejs klas tworzonych, który będzie implementowany przez konkretne rodzaje podklas oraz interfejs do tworzenia tych obiektów umożliwiający klasom dziedziczącym podejmowanie decyzji, której klasy instancja zostanie zwrócona. Jeśli brzmi to skomplikowanie lub niejasno za chwil kilka powinno być całkowicie zrozumiałe. Klasyczny już schemat wzorca metoda fabryki wygląda następująco:

{{< figure src="images/FactoryMethod.png" title="Schemat wzorca Factory Method" >}}

Powyższy rysunek został zaczerpnięty z historycznej już pracy "Design Patterns. Elements Of Reusable Object-Oriented Software", której autorami są Erich Gamma, Richard Helm, Ralph Johnson i John Vlissides, zwani w skrócie Bandą Czworga (Gang of Four). Uczestnicy schematu to:

* `Product` – definiuje interfejs obiektów tworzonych przez metodę wytwórczą. Pod pojęciem interfejsu rozumiemy klasę abstrakcyjną, klasę lub interfejs;
* `ConcreteProduct` – implementuje interfejs obiektów tworzonych przez metodę wytwórczą;
* `Creator` – twórca. Deklaruje metodę wytwórczą. Może zawierać domyślną implementację;
* `ConcreteCreator` – Konkretny twórca. Przedefiniowuje metodę wytwórczą.

Podobne lub niemal identyczne definicje znajdują się w każdej publikacji dotyczącej metody fabryki. Dzisiaj jednak próżno szukać w kodach źródłowych takiego nazewnictwa. W międzyczasie bowiem wykształciły się pewne konwencje, o których za chwilę.

## Stosowalność

Metody fabryki używamy jeśli:

* Nie możemy przewidzieć jakiego rodzaju obiekty powstaną. Wiemy jedynie kiedy chcemy je utworzyć;
* Potrzebujemy się pozbyć odpowiedzialności za tworzenie obiektów;
* Podczas tworzenia obiektów chcemy wykonać dodatkowe operacje.

## Implementacje

Możemy zdefiniować dwie główne implementacje wzorca:

* Fabryka jest klasą abstrakcyjną, która wymaga utworzenia podklas;
* Fabryka jest klasą konkretną, która może dopuszczać lub nie przedefiniowanie metody wytwórczej. Może być klasą statyczną.

Chciałbym zwrócić uwagę, iż zaczynamy powoli wchodzić w świat języka C# i platformy .NET, dlatego też należy mieć na uwadze, iż w innych językach programowania pewne elementy mogą być inaczej implementowane.

Ważną konsekwencją zastosowania klasy abstrakcyjnej oraz podklas jest to, iż użytkownicy muszą wiedzieć, jaką konkretną podklasę z metodą fabryki utworzyć. Rozwiązaniem tego problemu może być sparametryzowanie metody wytwórczej.

## Konwencje nazewnicze

Zgodnie z wcześniejszą zapowiedzią czas wspomnieć o konwencjach nazewniczych. Otóż przyjęło się, iż nazwy klas tworzących obiekty budujemy według schematu:

* \<nazwa_bazowa_intefejsu_obiektu_tworzonego\>`Factory`

Nazwa metod tworzących instancje obiektów:

* `Create`
* `Create`\<nazwa_konkretnego_obiektu_tworzonego\>

## Przykłady

Zanim przejdziemy do przykładów kilka uwag. Po pierwsze nigdzie nie jest powiedziane, że tworzone implementacje dowolnego wzorca, nie tylko Factory Method, muszą być zgodne z podstawowym schematem definiującym dany wzorzec. Zawsze powinniśmy wykorzystywać te możliwości, które w danej sytuacji najbardziej nam odpowiadają. Nie zawsze też rozwiązanie musi nawet odpowiadać głównym założeniom danego wzorca. Czasem wystarczy fragment. Możemy wtedy mówić o refaktoryzacji do wzorców. Pamiętajmy, aby do tematu nie podchodzić w sposób doktrynalny.

### Przykład 1

Na początek modelowa implementacja wzorca. Mamy obiekt `Customer` reprezentujący klienta, dla którego chcemy obliczyć wartość dowolnego rabatu. Potrzebujemy kilku algorytmów (sposobów) określania wielkości rabatu. Konkretny obiekt implementujący daną metodę obliczającą będzie zwracany przez wybraną metodę fabryki.

{{< figure src="images/FactoryMethod_Sample1.png" title="Factory Method - Przykład 1" >}}

Zaczynamy od zdefiniowania interfejsu `IDiscountStrategy` dla obiektów obliczających rabat i tworzonych przez metodę wytwórczą. Będzie on zawierał jedną metodę o nazwie `Calculate`, która jako parametr będzie przyjmowała obiekt typu `Customer`, obliczała rabat i zwracała jego wartość w postaci liczby rzeczywistej. Klasy konkretyzujące ów interfejs to `SpecialDiscountStrategy` oraz `StandardDiscountStrategy`. Abstrakcyjna klasa zawierająca metodę fabryki, która umożliwi zwracanie konkretnego sposóbu obliczania rabatu, nazywa się `DiscountStrategyFactory`. Klasy potomne to `SpecialDiscountStrategyFactory` oraz `StandardDiscountStrategyFactory`. Proszę zwrócić uwagę na zastosowane konwencje nazewnicze. Są one zgodne z tym o czym wspominałem kilka chwil wcześniej. Przy okazji wprowadziliśmy niejawnie wzorzec strategii (ang. Strategy), którego zadaniem jest umożliwienie zbudowania rodziny algorytmów wykonujących określone zadanie i używanie ich wymiennie.

```csharp
            Customer customer = new Customer();

            DiscountStrategyFactory factory =
                new StandardDiscountStrategyFactory();
            IDiscountStrategy discount = factory.Create();
            double value = discount.Calculate(customer);
```

Jak widzimy w przykładzie, wpierw tworzymy obiekt `StandardDiscountStrategyFactory` zawierający konkretną metodę fabryki. Następnie pobieramy obiekt `StandardDiscountStrategy` typu `IDiscountStrategy` obliczający rabat dla naszego klienta. Zmiana sposobu obliczania rabatu polega jedynie na zmianie `StandardDiscountStrategyFactory` na `SpecialDiscountStrategyFactory` pozostawiając resztę kodu bez zmian.

Konsekwencją powyższej implementacji jest mnogość podklas zawierających konkretną metodę fabryki zwracającą wybraną instancję tworzonego obiektu. Świadomie nie używam tutaj słowa wada - decyzja o tym, którą implementację metody fabryki wybierzemy powinna zależeć zawsze od założeń dotyczących tworzonego rozwiązania. Nie ma jednej idealnej implementacji.

### Przykład 2

Problem z nadmierną ilością podklas zawierających konkretną metodę wytwórczą można rozwiązać np. poprzez zastosowanie jednej klasy statycznej zawierającej wszystkie możliwe warianty metody fabryki. Wadą tego rozwiązania jest to, iż likwidujemy możliwość rozbudowy fabryki z wykorzystaniem dziedziczenia. Może mieć to zwłaszcza znaczenie jeśli pakiet zawierający fabrykę ma być zamknięty dla modyfikacji.

{{< figure src="images/FactoryMethod_Sample2.png" title="Factory Method - Przykład 2" >}}

Założenia są identyczne jak w poprzednim przykładzie. Poniżej dwa warianty kodu wykorzystującego proponowaną implementację:

```csharp
            Customer customer = new Customer();

            IDiscountStrategy discount =
                DiscountStrategyFactory.CreateStandardDiscountStrategy();
            double value = discount.Calculate(customer);
```

oraz

```csharp
            Customer customer = new Customer();

            StandardDiscountStrategy discount =
                DiscountStrategyFactory.CreateStandardDiscountStrategy();
            double value = discount.Calculate(customer);
```

Ze względu na czystość kodu drugie rozwiązania nie jest zalecane, nie mniej jednak można je stosować i dlatego też dla porządku je podaję. Zauważmy, iż w implementacji tej programista korzystający z klasy statycznej `DiscountStrategyFactory` zna wszystkie możliwe sposoby obliczania rabatów dla obiektu typu `Customer` poprzez proste odczytanie nazw metod wytwórczych dostępnych w fabryce.

### Przykład 3

Oczywiście nie zawsze chcemy, aby programista podejmował decyzje, którą instancję klasy zawierającej metodę wytwórczą będzie używał. Lub też przyjęte założenia uniemożliwiają wybranie rodzaju fabryki w prosty sposób. Wówczas możemy wybrać taką implementację wzorca Factory Method, w której stosowne decyzje podejmuje sama metody wytwórcza.

Jedno z rozwiązań prezentuje poniższy schemat:

{{< figure src="images/FactoryMethod_Sample3.png" title="Factory Method - Przykład 3" >}}

Przykład wykorzystania omawianej implementacji:

```csharp
            Customer customer = new Customer();

            IDiscountStrategy discount =
                DiscountStrategyFactory.Create(customer);
            double value = discount.Calculate(customer);
```

Tutaj programista zleca statycznej klasie zawierającej statyczną metodę fabryki utworzenie właściwego obiektu implementującego strategię udzielania rabatu. Decyzja podejmowana jest przez metodę wytwórczą na podstawie przekazywanego parametru typu `Customer`. W jaki sposób następuje podejmowanie decyzji, który obiekt obliczający zwrócić? To zależy od nas. Może to być na podstawie przypisania klienta do jakieś grupy klientów. Można wykorzystać parametry przekazywanego obiektu. Co tylko przyjdzie nam do głowy.

Pytanie czemu fabryka w powyższym przykładzie nie może od razu obliczać rabatu? Ot choćby dlatego, iż nie należy mieszać zakresów odpowiedzialności klas. Jeśli klasa ma za zadanie tworzyć instancje obiektów to nie powinna jednocześnie wykonywać np. operacji bazodanowych. W przeciwnym razie nasz kod bardzo szybko przestałby być łatwo testowalny, modyfikowalny i w niedługim okresie poprawnie działający.

### Przykład 4

Wraz z wersją drugą platformy .NET otrzymaliśmy możliwość skorzystania z typów **generic**. Dzięki temu przykład 1 oraz 2 możemy uprościć stosując statyczną klasę zawierającą pojedynczą metodę wytwórczą, która jako parametr typu może przyjmować implementacje interfejsu `IDiscount`.

{{< figure src="images/FactoryMethod_Sample4.png" title="Factory Method - Przykład 4" >}}

Uzyskany kod jest bardzo elastyczny. Zakładamy przy tym, iż programista decyduje o rodzaju rabatu w danym miejscu programu:

```csharp
            Customer customer = new Customer();

            IDiscountStrategy discount = DiscountStrategyFactory.
                Create<StandardDiscountStrategy>();
            double value = discount.Calculate(customer);
```

Najprostsza z możliwych implementacji metody fabryki wykorzystuje refleksję do tworzenia instancji klas. W przypadku dużej liczby klas, które mają być tworzone i mają konstruktor domyślny lub konstruktor z takimi samymi typami parametrów etc. poniższe rozwiązanie automatyzuje tworzenie instancji klas danego typu przez co zaoszczędzamy cenny czas (ale tylko ze względu na proces wpisywania powtarzającego się wielokrotnie kodu. Refleksja sama w sobie jest bardzo czasożerna i należy ją używać z rozwagą).

```csharp
    public static class DiscountStrategyFactory
    {
        public static TDiscount Create<TDiscount>()
            where TDiscount : IDiscountStrategy, new()
        {
            return Activator.CreateInstance<TDiscount>();
        }
    }
```

W celu zabezpieczenia się przed wywołaniem metody wytwórczej z niewłaściwymi parametrami nakładamy na metodę `Create` więzy, iż może ona być wywoływana z typem, który implementuje `IDiscountStrategy` oraz posiada publiczny i bezparametrowy konstruktor.

### Przykład 5

Na koniec trochę bardziej skomplikowany przykład, który pokaże nam w jaki sposób wykorzystać klasy typu **generic** do zbudowania Factory Method, która będzie zwracać właściwą instancję obiektu na podstawie przekazanego typu interfejsu, który zwracany obiekt musi implementować.

{{< figure src="images/FactoryMethod_Sample5.png" title="Factory Method - Przykład 5" >}}

W tej implementacji znowu przekazujemy podjęcie decyzji o rodzaju konkretnego obiektu, który zostanie zwrócony do metody wytwórczej. Aczkolwiek inaczej niż w przykładzie numer 3 decyzja będzie podejmowana na podstawie interfejsu, który ma być zwrócony a nie na podstawie cech obiektu typu `Customer`:

```csharp
            Customer customer = new Customer();

            ICustomerRepository repository = RepositoryFactory.
                Create<ICustomerRepository>();
            repository.Add(customer);
```

Podstawowa implementacja metody fabryki:

```csharp
    public static class RepositoryFactory
    {
        public static TRepository Create<TRepository>()
            where TRepository : IRepository
        {
            IRepository repository = default(IRepository);

            if (typeof(TRepository) ==
                typeof(ICustomerRepository))
                repository = new CustomerRepository();
            if (typeof(TRepository) ==
                typeof(IProductRepository))
                repository = new ProductRepository();

            return (TRepository)repository;
        }
    }
```

Jak widzimy wykorzystanie tego sposobu jest identyczne jak w poprzednich przykładach. Sama implementacja metody wytwórczej polega jedynie na sprawdzeniu jaki interfejs został przekazany a następnie budowana jest instancja klasy implementującej ów interfejs. Podobnie jak we wcześniejszych przykładach, tego typu sprawdzanie i podejmowanie decyzji może być bardziej skomplikowane.

Jeśli umożliwimy przed pierwszym wykorzystaniem metody `Create` klasy `RepositoryFactory` rejestrowanie w tej klasie dozwolonych interfejsów oraz powiązanych z nimi typów, które mają być tworzone, to `RepositoryFactory` będzie również implementować wzorzec, który nazywamy Service Locator. Możemy również zastosować zewnętrzne biblioteki i aplikacje, które dostarczają standardowych fabryk zdejmujących z nas konieczność pisania kodu tworzącego instancje klas. Konfiguracja tego typu rozwiązań oparta jest o pliki xml lub o atrybuty, którymi opatrujemy tworzone obiekty. Te zewnętrzne rozwiązania określamy bardzo często pojęciami:

* Inversion Of Control – pobieranie obiektów z zewnętrznych zasobów;
* Dependency Injection – tworzenie instancji zlecamy kontenerowi znającemu zależności pomiędzy klasami.

## Zakończenie

Wzorzec Factory Method jest jednym z najważniejszych wzorców kreacyjnych wykorzystywanych w dzisiejszym programowaniu. Należy jedynie pamiętać o właściwym wykorzystywaniu i rozsądku zwłaszcza jeśli stosujemy programowanie sterowane testami (Test-Driven Development).

Pliki do artykułu:

* [zine.net.pl](http://zine.net.pl/files/folders/673/download.aspx) lub [wg-net](http://wg-net.googlegroups.com/web/20071213_XVI_AW_FactoryMethod_pptx.zip) - prezentacja;
* [zine.net.pl](http://zine.net.pl/files/folders/674/download.aspx) lub [wg-net](http://wg-net.googlegroups.com/web/20071213_XVI_AW_FactoryMethod_src.zip) - kody źródłowe.