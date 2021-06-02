---
title: Wyjątki
date: 2007-05-15T13:19:00+02:00
languageCode: pl-pl
description: Testowanie poprawności rzucania wyjątkami w .NET C#
categories:
  - Development
tags:
  - .NET
  - Software testing
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/05/15/wyj-tki.aspx
disqus_disable: true
---

Jednym z elementów podlegających testowaniu jest sprawdzanie zachowania kodu w przypadku otrzymania nieprawidłowych danych. Reakcją na tego typu zdarzenia może być wyrzucanie wyjątków. Czasem jednak mogą powstać z tego tytułu pewne komplikacje. Załóżmy, że mamy do przetestowania poniższą klasę:

```csharp
    class CommandLibraryLocator : ICommandLibraryLocator
    {
        private ILibraryRepository libraryRepository;

        public CommandLibraryLocator(ILibraryRepository libraryRepository)
        {
            if (libraryRepository == null)
                throw new ArgumentNullException("libraryRepository");

            this.libraryRepository = libraryRepository;
        }

        public string GetPath(UserIdentity identity)
        {
            if (identity == null)
                throw new ArgumentNullException("identity");

            ...

            return path;
        }
    }
```

I rozważmy przykładowy kod testujący:

```csharp
        [Test]
        [ExpectedException(typeof(ArgumentNullException))]
        public void NullUserIdentityThrowsArgumentNullException()
        {
            CommandLibraryLocator locator =
                new CommandLibraryLocator(libraryRepository);
            locator.GetPath(null);
        }
```

Naszą intencją jest sprawdzenie czy instancja klasy `CommandLibraryLocator` zwróci wyjątek `ArgumentNullException` w przypadku wywołania metody `GetPath` z pustym parametrem typu `UserIdentity`. Jeśli zmienna przekazywana w konstruktorze klasy będzie z jakiś powodów `null` (te powody to oczywiście popełniony przez nas błąd), środowisko testowe poinformuje nas, iż test ten zakończył się poprawnie, mimo iż tak naprawdę wyjątek zwrócił nam konstruktor nie zaś metoda `GetPath`.

Jak się przed tego typu pomyłkami ustrzec? Otóż od wersji 2.4 środowisko testowe [NUnit](http://nunit.org) udostępnia nam bardzo rozbudowaną wersję atrybutu `ExpectedException`, który pozwala nam, poza typem oczekiwanego wyjątku, określić również komunikat, który spodziewamy się otrzymać w ramach tegoż wyjątku. Dzięki temu, możemy zmodyfikować powyższy test do następującej postaci:

```csharp
        [Test]
        [ExpectedException(typeof(ArgumentNullException),
            ExpectedMessage = "identity", MatchType = MessageMatch.Contains)]
        public void NullUserIdentityThrowsArgumentNullException()
        {
            CommandLibraryLocator locator =
                new CommandLibraryLocator(libraryRepository);
            locator.GetPath(null);
        }
```

Typ wyliczeniowy `MessageMatch` określa sposób interpretacji zwróconego komunikatu. Wywoływany w ramach metody `GetPath` wyjątek `ArgumentNullException` zwraca łańcuch `identity` jako część komunikatu, stąd też informuję środowisko testowe, iż spodziewana przeze mnie informacja będzie jedynie zawarta w komunikacie wyjątku.

Powyższe możliwości przydają się nie tylko z powodu takiego, iż możemy popełnić w czasie pisania testów błąd. Przyjrzyjmy się dla przykładu poniższej metodzie:

```csharp
        public ProcessResult Execute(UserIdentity identity, string name,
            string parameters, out string answer)
        {
            if (identity == null)
                throw new ArgumentNullException("identity");
            if (string.IsNullOrEmpty(identity.Name))
                throw new InvalidOperationException("identity.Name");
            if (string.IsNullOrEmpty(identity.LibraryDirectory))
                throw new InvalidOperationException("identity.LibraryDirectory");
            if (string.IsNullOrEmpty(identity.DownloadDirectory))
                throw new InvalidOperationException("identity.DownloadDirectory");
            if (string.IsNullOrEmpty(identity.UploadDirectory))
                throw new InvalidOperationException("identity.UploadDirectory");

            ...

            return ProcessResult.OK;
        }
```

Jak widzimy, bez możliwości określenia jaki komunikat spodziewamy się otrzymać, przetestowanie powyższego kodu bez wprowadzania własnych wyjątków nie będzie możliwe. Korzystając z nowych możliwości atrybutu `ExpectedException` testy te będą wyglądały następująco (poniżej tylko kilka pierwszych sprawdzeń):

```csharp
        [Test]
        [ExpectedException(typeof(InvalidOperationException),
            ExpectedMessage = "identity.Name",
            MatchType = MessageMatch.Contains)]
        public void EmptyUserIdentityNameThrowsInvalidOperationException()
        {
            UserIdentity user = new UserIdentity();

            ExecuteCommandProcess process = new ExecuteCommandProcess();
            process.Execute(user, name, parameters, out answer);
        }

        [Test]
        [ExpectedException(typeof(InvalidOperationException),
            ExpectedMessage = "identity.LibraryDirectory",
            MatchType = MessageMatch.Contains)]
        public void NullUserIdentityLibraryDirectoryThrowsInvalidOperationException()
        {
            identity.LibraryDirectory = null;

            ExecuteCommandProcess process = new ExecuteCommandProcess();
            process.Execute(identity, name, parameters, out answer);
        }
```

I to na tyle. Zapraszam do eksperymentów. Pardonsik... do testów oczywiście.