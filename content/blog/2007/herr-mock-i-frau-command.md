---
title: Herr Mock i Frau Command
date: 2007-06-21T17:01:00+02:00
lastmod: 2018-09-17T14:37:00+02:00
languageCode: pl-pl
description: Testowanie poleceń Command we wzorcu Model-View-Presenter
categories:
  - Development
tags:
  - .NET
  - Design patterns
  - Software testing
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/06/21/herr-mock-i-frau-command.aspx
disqus_disable: true
---

Sławetny wzorzec Model-View-Presenter ma swoje zalety, ale ma i swoje uciążliwości. Jedna z wykorzystywanych przeze mnie implementacji tego wzorca zakłada, iż widok będzie posiadał funkcję umożliwiającą dodawanie do menu widoku kolejnych poleceń, które będą zawierały metody zwrotne wywoływane jako reakcja w przypadku wykonania polecenia. Jest to o tyle konieczne, iż np. formularz wyświetlający tabelę może zachowywać się różnie (obsługiwać różne zestawy poleceń) w zależności od danych, które ma zaszczyt prezentować. Weźmy pod uwagę fragment prezentera umożliwiającego identyfikację użytkownika.

```csharp
        public void Initialize()
        {
            view.AddActionCommand(new Command("Zaloguj",
                this.Authenticate));
            view.AddActionCommand(new Command("Zamknij",
                this.Cancel));
        }

        private void Authenticate()
        {
        }

        private void Cancel()
        {
        }
```

Do widoku, do hipotetycznego menu Akcja, dodaję dwa polecenia. W jaki sposób widok to przetworzy w chwili obecnej nas nie interesuje. Problem w tym, iż metody wywoływane jako reakcja na wybrane przez użytkownika zdarzenia są prywatne. Oczywiście mogę je uczynić publicznymi, albo chronionymi w celu umożliwienia testowania (dziedziczenie prezentera), ale takie rozwiązanie nie przekonuje mnie. Nie chcę udostępniać nic co nie powinno być widoczne.

Rozwiązanie problemy zależy oczywiście od tego z jakich narzędzi korzystamy. W tym projekcie mogłem sobie na szczęści pozwolić na korzystanie z dobrodziejstw [Rhino Mocks](https://github.com/ayende/rhino-mocks). Wykorzystałem więc możliwości funkcji [Do](http://www.ayende.com/Wiki/Rhino+Mocks+The+Do()+Handler.ashx), która umożliwia zastąpienie wywołania metody własnym kodem. Zadeklarowałem w związku z tym grzecznie typ wymaganej akcji:

```csharp
        public delegate void AddActionCommandDelegate(Command command);
```

A następnie wykonałem pierwsze podejście:

```csharp
            AuthenticatePresenter presenter = new AuthenticatePresenter(
                service, controller, view);
            List<Command> commands = new List<Command>();

            view.AddActionCommand(null);
            LastCall.IgnoreArguments().Do(
                new AddActionCommandDelegate(delegate(Command command)
                {
                    commands.Add(command);
                })).Repeat.Any();
            mocks.ReplayAll();

            presenter.Initialize();
            commands[0].Execute();

            mocks.VerifyAll();
```

Zadziałało, ale nie do końca mnie usatysfakcjonowało. Dowolnie duża liczba widoków i prezenterów oznaczać będzie powtórzenia kodu jeśli chodzi o tworzenie listy poleceń i wywoływanie własnego kodu. Znacznie lepszym podejściem wydawało mi się stworzenie klasy pomocniczej, której zastosowanie mogłoby uprościć kod do następującego:

```csharp
            AuthenticatePresenter presenter = new AuthenticatePresenter(
                service, controller, view);
            CommandMockHelper helper = new CommandMockHelper();

            view.AddActionCommand(null);
            LastCall.IgnoreArguments().Do(
                new CommandMockHelper.AddActionCommandDelegate(
                helper.AddActionCommand)).Repeat.Any();

            mocks.ReplayAll();

            presenter.Initialize();
            helper.Commands[0].Execute();

            mocks.VerifyAll();
```

A oto klasa pomocnicza. Jej zadaniem jest stworzenie listy dodanych do widoku poleceń, w celu łatwego wywoływania ich w czasie testów.

```csharp
    class CommandMockHelper
    {
        public delegate void AddActionCommandDelegate(Command command);

        public List<Command> Commands = new List<Command>();

        public void AddActionCommand(Command command)
        {
            Commands.Add(command);
        }
    }
```

Oczywiście prezentowane przykłady testów są maksymalnie uproszczone. Rzeczywisty kod powinien zawierać co nieco przed `mocks.ReplayAll()`.