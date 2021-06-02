---
title: Implementacja Inversion of Control - wersja 1.1
date: 2009-01-27T12:38:00+01:00
languageCode: pl-pl
description: Własna implementacja kontenera Inversion of Control w .NET C#
categories:
  - Development
tags:
  - .NET
  - .NET Compact Framework
  - Design patterns
  - Mobile
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/01/27/implementacja-inversion-of-control-wersja-1-1.aspx
disqus_disable: true
---

Od [ostatniej notki]({{< ref "implementacja-inversion-of-control" >}}) opisującej wykorzystywany przeze mnie własnej produkcji kontener IoC wprowadziłem kilka modyfikacji czyniących rozwiązanie bardziej elastycznym, ale wciąż pozostające wierne podstawowym założeniom:

* Wydajne i łatwe w użyciu;
* Zminimalizowane użycie refleksji;
* Brak plików konfiguracyjnych.

## Czymże jest kontener IoC

Kontener IoC umożliwia programiście wprowadzenie w aplikacji luźnych powiązań pomiędzy obiektami. Programista rejestruje interfejsy i klasy abstrakcyjne wraz z typami implementującymi, instancjami lub procedurami tworzącymi instancje klas na żądanie:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>();
```

Możliwa jest również rejestracja typów, które mogą posiadać instancje:

```csharp
            ObjectLocator.Register<Database>();
```

Zwracany w czasie rejestracji interfejs `IObjectProfile` korzysta ze wzorca **Fluent Interface** w celu właściwej konfiguracji:

```csharp
            ObjectLocator.Register<IContactRepository>().
                AsSingleton().
                WithType<ContactRepository>().
                AndBuildUp(new object[] { "test" });
```

Skorzystanie z obiektu zarejestrowanego w kontenerze polega na wywołaniu metody `GetInstance` z ewentualnymi parametrami, które zostaną przekazane do konstruktora:

```csharp
            var repository = ObjectLocator.GetInstance<IContactRepository>();
```

W przeciwieństwie do rozwiązań typu **Dependency Injection** w opisywanym kontenerze IoC nie jest możliwe skorzystanie z automatycznego uzupełniania parametrów konstruktorów czy wstrzykiwania instancji do właściwości.

## Zmiany

W porównaniu do wersji wcześniejszej, fasada rozwiązania, statyczna od teraz klasa `ObjectLocator` zwraca nam zamiast obiektu `ObjectProfile` interfejs `IObjectProfile`. Z punktu widzenia programisty korzystającego z IoC jest to najważniejsza zmiana. Może to bowiem oznaczać konieczność zmiany między innymi definicji metod zwrotnych wywoływanych w czasie tworzenia instancji, jeśli oczywiście z takowych korzystamy, na:

```csharp
        Action<IObjectProfile<TRegisteredAs>, TRegisteredAs>
```

oraz

```csharp
        Func<IObjectProfile<TRegisteredAs>, object[], TRegisteredAs>
```

Są to jednak zmiany na poziomie wręcz kosmetycznym. Jednym słowem nie sprawią problemów.

Wewnętrznie `ObjectLocator` deleguje teraz wszystkie zadania do implementacji interfejsu `IObjectContainer`. Dzięki temu korzystanie z `ObjectLocator` nie determinuje nam konkretnej implementacji kontenera. Innymi słowy wykorzystanie tego rozwiązania nie przekreśla możliwości skorzystania z innego rozwiązania **Inversion of Control** lub nawet **Dependency Injection**. Wystarczy bowiem wówczas zaimplementować interfejs `IObjectProfile` w celu konfiguracji rejestrowanego typu oraz interfejs `IObjectContainer` do zarządzania rejestrowanymi typami. Podmiana kontenera oznacza przypisanie klasie `ObjectLocator` poprzez właściwość `Container` interesującej nas instancji implementującej `IObjectContainer`.

{{< figure src="images/IoC_1_1.png" title="Implementacja Inversion of Control" >}}

Dzięki wprowadzeniu interfejsu `IObjectContainer` prostsze stało się korzystanie z kontenera w pakietach (**assemblies**) zewnętrznych. Do tej pory pobieranie instancji oznaczało konieczność odwoływania się do statycznej klasy `ObjectLocator`. Teraz wystarczy przekazać do pakietu zewnętrznego zmienną typu `IObjectContainer`.

Łatwiejsze stało się również testowanie aplikacji korzystających z rozwiązania IoC. Nie trzeba teraz przed każdym testem konfigurować klasy `ObjectLocator` poprzez rejestrowanie typów etc. Zamiast tego wystarczy obecnie przypisać do właściwości `ObjectLocator.Container` testową implementację kontenera. Poniżej przykład takiego kodu wykorzystującego bibliotekę [Rhino.Mocks](http://ayende.com/projects/rhino-mocks.aspx):

```csharp
            var container = MockRepository.GenerateStub<IObjectContainer>();
            var repository = new CustomerRepository();
            container.Stub(
                x => x.GetInstance<IRepository<Customer>>()).
                Return(repository);
            ObjectLocator.Container = container;
            Assert.AreEqual(
                repository,
                ObjectLocator.GetInstance<IRepository<Customer>>());
```

## Nowości

Poza opisaną już właściwością `Container`, w klasie `ObjectLocator` i interfejsie `IObjectContainer` pojawiła się metoda `Contains` umożliwiająca sprawdzenie czy dany typ został już zarejestrowany. Jest to cenna możliwość zwłaszcza jeśli potrzebujemy dla danego odbiorcy (użytkownika) oprogramowania przerejestrować konkretny typ. W wersji wcześniejszej konieczne było sprawdzenie czy istnieje profil dla danego typu wykorzystując metodę `GetProfile`.

Na poziomie konfigurowania typu (interfejs `IObjectProfile`) dodano możliwość zarejestrowania parametrów domyślnych przekazywanych do konstruktora nowego obiektu. W poniższym przykładzie parametr ten jest pobierany z kontenera IoC:

```csharp
            ObjectLocator.Register<IContactRepository>().
                WithType<ContactRepository>().
                WithParameters(new object[]
                                   {
                                       ObjectLocator.GetInstance<Database>()
                                   }).
                AsSingleCall();
```

Parametry domyślne mogą być zastępowane nowymi w czasie wywołań metody `GetInstance`.

## Podsumowanie

Opisywane rozwiązania używam z powodzeniem od ponad kilkunastu miesięcy w [projekcie]({{< ref "rozwiazanie-mobilne" >}}), którego części składowe działają na urządzeniach mobilnych. Udostępnione kody są oparte na nowej licencji BSD co umożliwia używanie opisywanego kontenera IoC w projektach komercyjnych. Kody źródłowe zawierają testy jednostkowe, które umożliwiają dokładniejsze zapoznanie się z możliwościami rozwiązania.

Pliki do artykułu:

* [IoC](http://zine.net.pl/files/folders/2795/download.aspx) - kody źródłowe.