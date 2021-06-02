---
title: Implementacja Inversion of Control
date: 2008-08-24T11:49:00+02:00
lastmod: 2018-09-17T13:37:00+02:00
languageCode: pl-pl
description: Własna implementacja kontenera Inversion of Control w .NET C#
categories:
  - Development
tags:
  - .NET
  - .NET Compact Framework
  - Design patterns
  - Mobile
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/08/24/implementacja-inversion-of-control.aspx
disqus_disable: true
---

Jedną z cech dobrego oprogramowania są luźne powiązania pomiędzy klasami. Droga do tego celu ciężka i kręta. Bez dwóch zdań. Wśród technik i wzorców, które należy w tym celu stosować znajdują się fabryki (Factory) oraz lokalizatory usług (Service Locator), dzięki którym tworzeniem instancji obiektów czy implementacji zadanych interfejsów zajmują się wyspecjalizowane klasy. Z [tworzeniem obiektów]({{< ref "tworzenie-obiektow" >}}), w kontekście wymienionych powyżej praktyk, związane są następujące koncepcje:

* Inversion of Control - instancje klas pobierane są z zewnętrznych zasobów;
* Dependency Injection - tworzenie instancji zleca się zewnętrznemu obiektowi (kontenerowi) znającemu zależności pomiędzy właściwymi klasami.

Programista .NET chcący skorzystać z darmowych produktów IoC i DI ma do wyboru [wiele rozwiązań](http://www.hanselman.com/blog/ListOfNETDependencyInjectionContainersIOC.aspx). Przykładem może być [StructureMap](http://structuremap.github.io), którego autorem jest Jeremy Miller (prowadzi nota bene bardzo ciekawy [blog](https://jeremydmiller.com)), czy też [Unity](https://github.com/unitycontainer/unity) firmy Microsoft. W przypadku platformy .NET Compact Framework sytuacja nie wygląda już tak radośnie. Miesiące temu, kiedy poszukiwałem bezpłatnego kontenera mogącego działać w ramach aplikacji mobilnych, jedynym projektem był Mobile ObjectBuilder z [Mobile Client Software Factory](http://www.codeplex.com/smartclient), który podobnie jak w bibliotekach przeznaczonych na platformę .NET do oznaczania relacji pomiędzy klasami wykorzystywał atrybuty. Niestety rozwiązanie to nie satysfakcjonowało mnie. Między innymi z powodu wydajności.

Z braku istniejących darmowych produktów przeznaczonych na platformę .NET Compact Framework postanowiłem zaimplementować własne rozwiązanie typu Inversion of Control. Miało ono spełniać następujące kryteria:

* Być wydajne i łatwe w użyciu;
* Używać jak najmniej refleksji;
* Nie używać plików konfiguracyjnych.

Punkt pierwszy jest jasny i oczywisty. Pozostałe dwa założenia są konsekwencją znacznie mniejszych możliwości urządzeń mobilnych w zakresie mocy obliczeniowych czy zasobów pamięciowych w porównaniu z komputerami typu desktop, na których uruchamiana jest pełna wersja .NET. Ponieważ rozwiązanie przeznaczone było dla .NET Compact Framework, w projekcie nie zostały zupełnie uwzględnione takie kwestie jak wielowątkowość czy możliwość działania w ramach środowiska ASP.NET.

Przy tworzeniu kontenera założyłem, iż programista jest podmiotem działania a nie przedmiotem (wiem co mówię, bo jestem politologiem). Innymi słowy założyłem, iż jest inteligentny. Stąd duża elastyczność przy konfigurowaniu rejestrowanego typu i do minimum ograniczone szukanie absurdów konfiguracyjnych. Zresztą w tym przypadku według mnie takie podejście jest z korzyścią dla czytelności i przejrzystości kodu.

Efektem prac była biblioteka IoC nazwana dla uproszczenia... IoC. W zasobach do tego wpisu można znaleźć [kody źródłowe](http://zine.net.pl/files/folders/1807/download.aspx) stworzonego rozwiązania. W dalszej części artykułu przedstawię możliwości biblioteki i opiszę w jaki sposób można z niej korzystać.

Gwoli wyjaśnienia na koniec. Wcześniej w swoich projektach używałem [rozwiązania]({{< ref "dobre-praktyki-w-projektowaniu-aplikacji-mobilnych" >}}) opartego na wzorcu Service Locator.

## IoC

Powstałe rozwiązanie IoC, w dalszej części nazywane również kontenerem, składa się z trzech klas, z których tylko dwie są bezpośrednio dostępne programiście, i typu wyliczeniowego:

* `ObjectLocator` - główna klasa rozwiązania. Dostępna jako Singleton. Pozwala rejestrować typy i pobierać ich instancje;
* `ObjectProfile` - klasa umożliwiająca skonfigurowanie rejestrowanego typu. Oparta o wzorzec Fluent Interface;
* `ObjectCreator` - wewnętrzna klasa statyczna odpowiedzialna za tworzenie instancji zarejestrowanych typów;
* `LifetimeStyle` - typ wyliczeniowy. Dozwolone wartości czasu życia obiektu.

{{< figure src="images/IoC.png" title="Implementacja Inversion of Control" >}}

Rozpoczęcie pracy z kontenerem polega na zarejestrowaniu typu. W najprostszej formie będzie to zarejestrowanie klasy:

```csharp
            ObjectLocator.Register<ContactRepository>();
```

Oczywiście w tym przypadku klasa nie może być abstrakcyjna ponieważ nie można utworzyć instancji takiej klasy.

Aby móc zarejestrować interfejs, klasę abstrakcyjną czy też skonfigurować typ musimy skorzystać z bardziej wyrafinowanych metod. Aby uniknąć tworzenia wielu wersji metody rejestrującej wybrano wariant, w którym w czasie rejestracji przez metodę `Register` zwracany jest skojarzony z typem obiekt `ObjectProfile` umożliwiający konfigurację. Dzięki wzorcowi Fluent Interface (metody konfigurujące zwracają referencję do obiektu, który je zawiera) nie jest konieczne ustalanie poszczególnych właściwości tego obiektu. Wszystko można wykonać w czasie jednego wywołania. Zobaczmy więc jak wygląda rejestracja interfejsu:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>();
```

Parametrem metody `WithType` jest klasa, która implementuje rejestrowany typ. Parametr ten nie powinien być oczywiście klasą abstrakcyjną. W wywołaniu metody `Register` możliwe jest również skorzystanie z typów generic:

```csharp
            ObjectLocator.Register<IRepository<Contact>>().
                WithType<ContactRepository>();
```

Identycznie jest w przypadku rejestrowania klas abstrakcyjnych.

Domyślny czas życia obiektu w kontenerze to `LifetimeStyle.Singleton`, czyli wszystkie wywołania w kodzie będzie obsługiwała tylko jedna instancja klasy. Można oczywiście to zmienić tak, aby dla każdego wywołania tworzona była nowa instancja klasy:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>().
                AsSingleCall();
```

Aby pobrać z kontenera instancję musimy wywołać metodę `GetInstance` podając w wywołaniu zarejestrowany typ. Poniżej wywołanie dla przypadku, kiedy zarejestrowaliśmy wcześniej interfejs:

```csharp
            var repository = ObjectLocator.GetInstance<IContactRepository>();
```

Oczywiście konstruktor obiektu, którego instancję tworzymy może zawierać parametry. Dla takich przypadków korzystamy z przeciążonej metody `GetInstance`:

```csharp
            var repository = ObjectLocator.GetInstance<IContactRepository>(
                new object[] { new Database() });
```

W powyższym przykładzie przy tworzeniu obiektu korzystamy z konstruktora, który zawiera jeden parametr. Jeśli typ `Database` zostałby wcześniej zarejestrowany w kontenerze, możemy do konstruktora przekazać typ pobrany z IoC:

```csharp
            var repository = ObjectLocator.GetInstance<IContactRepository>(
                new object[] { ObjectLocator.GetInstance<Database>() });
```

Wszystkie pokazane do tej pory wywołania metody `GetInstance` skutkowały utworzeniem instancji z wykorzystaniem refleksji, co nie zawsze jest dobrym rozwiązaniem. Dodatkowo wewnętrzna, statyczna klasa `ObjectCreator` może nie poradzić sobie w sytuacji kiedy klasa, której instancję chcemy utworzyć, posiada kilka konstruktorów o takiej samej liczbie parametrów i niektóre wartości przekazywane w wywołaniu metody `GetInstance` mają wartość `null`. Rozwiązaniem jest taka konfiguracja rejestrowanego typu, aby tworzenie instancji odbywało się poza kontenerem. Najprostszy sposób widzimy poniżej:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithInstance(new ContactRepository());
```

Interfejs `IContactRepository` rejestrowany jest od razu z gotową instancją obiektu. Oczywiście powyższa konfiguracja ma sens tylko dla czasu życia obiektu `LifetimeStyle.Singleton` ponieważ przekazany obiekt będzie używany do obsługi wszystkich żądań.

Zamiast przekazywać do kontenera gotowy obiekt, można proces tworzenia opóźnić do momentu, kiedy dana instancja rzeczywiście będzie potrzebna. Do tego służy metoda `CallWhenCreating` klasy `ObjectProfile` która jako parametr przyjmuje metodę zwrotną typu:

```csharp
            Func<ObjectProfile<TRegisteredAs>, object[], TRegisteredAs>
```

Delegat `Fun` wprowadzony w .NET 3.5 i zawierający powyższe parametry odpowiada następującemu delegatowi z .NET 2.0:

```csharp
            delegate TRegisteredAs ObjectCreatingCallback<TRegisteredAs>(
                ObjectProfile<TRegisteredAs> profile, object[] parameters)
                where TRegisteredAs : class;
```

Jak widzimy metoda tworząca otrzymuje z kontenera obiekt zawierający ustawienia zarejestrowanego typu oraz parametry dla konstruktora. Użycie nie jest skomplikowane. Najpierw wywołanie wykorzystujące delegat anonimowy:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>().
                CallWhenCreating(
                delegate(
                    ObjectProfile<IContactRepository> profile,
                    object[] parameters)
                    {
                        var database = (Database)parameters[0];
                        return new ContactRepository(database);
                    });
```

I wyrażenie lambda, nowość w ramach platformy .NET 3.5:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>().
                CallWhenCreating((profile, parameters) =>
                    {
                        var database = (Database)parameters[0];
                        return new ContactRepository(database);
                    });
```

Jeśli tworząc nowy obiekt nie potrzebujemy informacji dotyczących konfiguracji typu oraz nie przekazujemy do konstruktora wartości możemy w przypadku delegata pominąć parametry:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>().
                CallWhenCreating(
                delegate
                    {
                        return new ContactRepository();
                    });
```

Ważna również jest możliwość wywołania zdefiniowanej metody zwrotnej po utworzenia obiektu. Przykładowo wskazane może być, aby dla każdego formularza w programie wstawić w tytule okna nazwę aplikacji, zdefiniować identyczne rozmiary itp. W celu zapewnienia spełnienia tych wymagań kontener zawiera metodę `CallWhenCreated` klasy `ObjectProfile` zadeklarowaną z następującym parametrem:

```csharp
            Action<ObjectProfile<TRegisteredAs>, TRegisteredAs>
```

Podobnie jak w przypadku metody zwrotnej wywoływanej w celu utworzenia obiektu, tak i tutaj delegat `Action` możemy przedstawić, jeśli chcielibyśmy korzystać z wcześniejszych wersji platformy .NET (zobacz uwagi na końcu notki), w postaci własnego delegata:

```csharp
            delegate void ObjectCreatedCallback<TRegisteredAs>(
                ObjectProfile<TRegisteredAs> profile, TRegisteredAs instance)
                where TRegisteredAs : class;
```

Tradycyjnie przykład zastosowania. Tym razem użyjemy zdefiniowanej metody zwrotnej:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>().
                CallWhenCreated(RepositoryCreated);
```

Na koniec procesu rejestracji możemy sobie zażyczyć, aby instancja danego typu została od razu utworzona. Tę właściwość możemy wykorzystywać w przypadku dużych klas, które na pewno będą używane w czasie działania aplikacji. Oczywiście ma to sens jedynie w przypadku tych klas, dla których tylko jedna instancja będzie obsługiwała wszystkie żądania w programie. Dobrym przykładem są tutaj formularze. Poniżej stosowny przykład:

```csharp
            ObjectLocator.Register<IContactRepository>().
                AsSingleton().
                WithType<ContactRepository>().
                AndBuildUp();
```

Oczywiście na tym etapie również istnieje możliwość przekazania do konstruktora klasy wymaganych wartości:

```csharp
            ObjectLocator.Register<IContactRepository>().
                AsSingleton().
                WithType<ContactRepository>().
                AndBuildUp(new object[] { "test" });
```

Metoda `AndBuildUp` w przeciwieństwie do pozostałych metod klasy `ObjectProfile` zwraca utworzoną instancję klasy a nie wskazanie na obiekt konfigurujący.

## W praktyce

Najwłaściwsze miejsce konfiguracji kontenera to start programu. Osobiście korzystam z fabryki abstrakcyjnej, której konkretne implementacje decydują o sposobie działania programu np.:

```csharp
    internal abstract class ApplicationInitializer
    {
        /// <summary>
        /// Rejestrowanie widoków.
        /// </summary>
        public abstract void RegisterViews();

        /// <summary>
        /// Rejestracja fabryk abstrakcyjnych.
        /// </summary>
        public abstract void RegisterFactories();

        /// <summary>
        /// Rejestracja usług - interfejsu pomiędzy modelem a prezentacją.
        /// </summary>
        public abstract void RegisterServices();

        /// <summary>
        /// Rejestracja repozytoriów - klas umożliwiających dostęp do danych.
        /// </summary>
        public abstract void RegisterRepositories();
    }
```

Zastosowany podział, czyli nazewnictwo i przeznaczenie metod wytwórczych, wynika głównie ze wzorca **Model-View-Presenter** wykorzystywanego z lubością przeze mnie w aplikacjach. Poniżej kilka przykładowych rejestracji widoków, gdzie formularze są w programie traktowane jako interfejsy:

```csharp
        public override void RegisterViews()
        {
            ObjectLocator.Register<IAuthenticateView>().
                WithType<AuthenticateForm>().
                AsSingleCall().
                CallWhenCreated(ViewCreated);
            ObjectLocator.Register<IDataGridView>().
                WithType<DataGridForm>().
                CallWhenCreated(ViewCreated).
                AndBuildUp();
            ObjectLocator.Register<IMessageView>().
                WithType<MessageForm>().
                CallWhenCreated(ViewCreated);
        }
```

Metoda `ViewCreated` pozwala na ustawienie parametrów wspólnych dla wszystkich formularzy takich jak widoczności wybranych kontrolek czy rozmiar. A tak wygląda rejestracja fabryk:

```csharp
        public override void RegisterFactories()
        {
            ObjectLocator.Register<BusinessEntityFactory>();
            ObjectLocator.Register<DataGridStyleBuilder>();
            ObjectLocator.Register<DocumentHtmlFormatterFactory>();
        }
```

W powyższym kodzie wszystkie fabryki są jednocześnie klasami implementującymi. Domyślnie obiekt tworzony jest w czasie pierwszego wywołania i istnieje przez cały czas pracy aplikacji. Zauważmy, iż dzięki takiemu podejściu jedynym singletonem w naszym kodzie pozostanie kontener IoC!

Usługi dostarczają do prezenterów model:

```csharp
        public override void RegisterServices()
        {
            ObjectLocator.Register<IAuthenticateService>().
                WithType<AuthenticateService>().
                AsSingleCall();
            ObjectLocator.Register<ICustomerService>().
                WithType<CustomerService>().
                AsSingleCall();
            ObjectLocator.Register<ISalesService>().
                WithType<SalesService>().
                AsSingleCall();
        }
```

W przeciwieństwie do fabryk instancje usług będą tworzone dla każdego wywołania.

Na koniec fragment odpowiadający za klasy umożliwiające dostęp do danych. Rejestracja powiązana jest z definicją delegatów odpowiedzialnych za tworzenie obiektów.

```csharp
        public override void RegisterRepositories()
        {
            ObjectLocator.Register<Database>().
                AsSingleton().
                CallWhenCreating(
                delegate
                    {
                        return new Database(connectionString);
                    });
            ObjectLocator.Register<ITransaction>().
                WithType<Transaction>().
                AsSingleCall().
                CallWhenCreating(
                delegate
                    {
                        return new Transaction(
                            ObjectLocator.Get<Database>());
                    });

            ObjectLocator.Register<ICustomerRepository>().
                WithType<CustomerRepository>().
                AsSingleCall().
                CallWhenCreating(
                delegate
                    {
                        return new CustomerRepository(
                            new CustomerDAO(
                                ObjectLocator.Get<Database>(),
                                ObjectLocator.Get<BusinessEntityFactory>(),
                                new PaymentMethodConverter()));
                    });
        }
```

## Zakończenie

Opisane powyżej rozwiązanie wykorzystuję z powodzeniem od bardzo dawna. I sprawdza się znakomicie. Udostępnione kody są oparte na nowej licencji BSD. Nie ma więc problemu aby, jeśli ktoś chce, wykorzystać zaprezentowaną przeze mnie implementację IoC we własnych rozwiązaniach - również komercyjnych. Z chęcią wysłucham również wszelkich uwag dotyczących przedstawionego rozwiązania.

Jeśli chodzi o wersję platformy .NET i .NET Compact Framework to rozwiązanie zostało oparte o wydanie 3.5. Osoby, które chciałyby wykorzystać opisany kontener we wcześniejszych wersjach muszą zastąpić wyrażenia lambda delegatami, oraz wywołania `Action` oraz `Fun` podanymi przeze mnie odpowiednikami własnych delegatów. Oczywiście z powodu wykorzystywanych typów generic kontener ten nie będzie działać na platformie .NET 1.1.

Pliki do artykułu:

* [IoC](http://zine.net.pl/files/folders/1807/download.aspx) - kody źródłowe.

PS. Michał Grzegorzewski ogłosił jakiś czas temu [konkurs](http://zine.net.pl/blogs/mgrzeg/archive/2008/06/25/msdnforopensourceprojects.aspx) na projekt Open Source z główną nagrodą w postaci MSDN w wersji Premium. Jeśli ktoś nie ma pomysłu na taki projekt to podpowiadam. Można spróbować rozbudować powyższe rozwiązanie o możliwość pracy w środowiskach wielowątkowych czy ASP.NET.