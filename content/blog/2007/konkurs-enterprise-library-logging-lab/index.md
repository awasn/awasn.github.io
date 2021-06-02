---
title: Konkurs Enterprise Library - Logging (LAB)
date: 2007-09-24T11:32:00+02:00
languageCode: pl-pl
description: Opis wymagań konkursu dotyczącego Microsoft Enterprise Library Logging Application Block
categories:
  - Development
tags:
  - .NET
  - Design patterns
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/09/24/konkurs-enterprise-library-logging-lab.aspx
disqus_disable: true
---

## Refaktoryzacja do Logging Application Block

Potrzebujemy możliwości śledzenia zachodzących w aplikacji zdarzeń. Ważne jest, aby informacje mogły być zapisywane w jak największej liczbie miejsc takich jak pliki, bazy danych czy poczta elektroniczna, a sama konfiguracja była niezależna od kodu programu.

Aby to osiągnąć korzystamy z możliwości Enterprise Library Logging Application Block w wersji 3.1 - May 2007.

## Uzasadnienie

Możliwość rejestracji zdarzeń (logging) zachodzących w aplikacji jest cechą bardzo pożądaną. Nie należy jednak przez zdarzenia rozumieć jedynie błędy czy problemy pojawiające się w czasie działania programu. W prostych aplikacjach gdzie logika jest nieskomplikowana, będzie to oczywiście główne źródło zainteresowań programistów, ale w bardziej rozbudowanych rozwiązaniach ważne będzie również w jakich stanach program się znajduje, jak następuje przechodzenie pomiędzy stanami, w jakiej kolejności są wywoływane elementy składowe aplikacji, jakie podejmowane czynności (akcje) etc. Jeśli mamy do czynienia z usługą czy warstwą biznesową aplikacji rozproszonej, generowany w czasie działania aplikacji log będzie dla osoby nadzorującej podstawowym, a często jedynym, źródłem informacji.

Korzystając z Microsoft Enterprise Library Logging Application Block możemy raportować zdarzenia zachodzące w programie w następujących ujściach:

* Dziennik zdarzeń (event log);
* Poczta elektroniczna;
* Baza danych;
* Kolejka komunikatów (message queue);
* Plik tekstowy;
* Zdarzenie WMI;
* Dowolne ujście użytkownika (poprzez własne klasy rozszerzające).

Każde rejestrowane zdarzenie może być opisane dużą liczbą informacji takich jak:

* Tytuł zdarzenia;
* Opis zdarzenia;
* Ważność (severity);
* Priorytet;
* Własne kategorie;
* Słownik dodatkowych właściwości (pary klucz-wartość);

Jednocześnie zapis uzupełniany jest między innymi:

* Nazwą komputera;
* Informacjami dotyczącymi domeny aplikacji (AppDomain);
* Informacjami dotyczącymi bieżącego procesu i wątku.

Zarządzanie rodzajem ujść (plik tekstowy, zdarzenie systemu etc.) oraz poziomem dokładności zdarzenia (informacje, ostrzeżenia, błędy etc.) odbywa się poprzez wpisy w plikach konfiguracyjnych aplikacji. Dopuszcza się również możliwość ustalania tych reguł na poziomie kodu źródłowego, aczkolwiek w tym przypadku, ze względu na przeznaczenie bloku, nie jest to zalecane postępowanie. Dzięki umiejscowieniu zarządzania logami w plikach konfiguracyjnych aplikacji, uniezależnia się logikę działania aplikacji od nadzorowania zachodzących w niej zdarzeń.

Należy wziąć pod uwagę, iż Logging Application Block korzysta w czasie działania z projektu ObjectBuilder, którego zadaniem jest tworzenie instancji obiektów w oparciu o Dependency Injection. We wzorcu tym klasy są tworzone przez kontener, który dla osiągnięcia celu korzysta z refleksji oraz atrybutów opisujących zależności pomiędzy klasami. Takie podejście oznacza zapotrzebowanie na dodatkowe zasoby komputera co wskazane jest zawsze brać pod uwagę. Notabene programiści Microsoft poprzez nazwanie rozwiązania Enterprise Library wskazują pośrednio właściwe przeznaczenie użytkowania tych bloków.

## Sposób wykonania

Czynności, które należy wykonać, w celu skorzystania z Logging Application Block są następujące:

* Dodajemy w ramach projektu referencję (polecenie **Add reference…**) do biblioteki Microsoft.Practices.EnterpriseLibrary.Logging;
* Dodajemy w klasach, w których chcemy skorzystać z możliwości biblioteki, przestrzeń nazw `Microsoft.Practices.EnterpriseLibrary.Logging`;
* W miejscach kodu, gdzie chcemy rejestrować zdarzenia wywołujemy przeciążoną (overload) metodę statyczną `Logger.Write`, do której przekazujemy informacje o zdarzeniu;
* Dokonujemy właściwych wpisów w pliku konfiguracyjnym aplikacji.

## Przykład

Refaktoryzacja do bloku Logging Application Block jest, w przeciwieństwie np. do Validation Application Block, wyjątkowo prosta. Polega ona tak naprawdę na wstawieniu metody rejestrującej zdarzenia w interesujących nas miejscach kodu. Jako przykład posłuży nam aplikacja Web, której zadaniem jest umożliwienie obliczania wyniku z dzielenia dwóch liczb całkowitych. Nasze rozwiązanie oprzemy o uchwyty HTTP – implementację interfejsu `IHttpHandler`, który umożliwia nam obsługę żądania opartego o protokół HTTP w ramach środowiska ASP.NET (istniejące już w .NET implementacje tego interfejsu to między innymi formularze Web oraz usługi sieciowe). Cały projekt składa się z następujących elementów:

* **Global.asax** – będący miejscem kontroli przechodzących żądań;
* **DivideHandler.ashx** – wykonujący właściwą operację dzielenia;
* **Web.config** – zawierający konfigurację aplikacji;
* **Resources.resx** – łańcuchy wyświetlane w ramach aplikacji.

Aby wykonać obliczenia należy wywołać **DivideHandler.ashx** podając jako parametry zapytania liczbą, którą chcemy podzielić oraz liczbę dzielącą np.: `http://localhost:1109/DivideHandler.ashx?a=8&b=2`.

Dołączony kod źródłowy aplikacji jest tak skonfigurowany, iż wywołanie właściwego żądania powinno nastąpić automatycznie zaraz po uruchomieniu. Do interakcji z przykładem wystarczy zwykła przeglądarka internetowa, przy czym polecam w tym wypadku Internet Explorer. Firefox w przypadku otrzymania statusu wystąpienia błędu zgłasza informacje o błędach parsowania XML.

Zakładamy, iż z naszego punktu widzenia ważne są następujące zdarzenia:

* Fakt wywołania uchwytu;
* Rezultat operacji;
* Błędy związane z parametrami np. z próbą dzielenia przez zero.

Najważniejszy kod naszej aplikacji, zawierający już wymagane wpisy rejestrujące zdarzenia, wygląda następująco:

```csharp
        /// <summary>
        /// Przetworzenia żądania klienta.
        /// </summary>
        /// <param name="context">Kontekst żądania.</param>
        public void ProcessRequest(HttpContext context)
        {
            Logger.Write(Properties.Resources.DividesTwoIntegerValues);

            int dividend = 0, divisior = 0;

            try
            {
                ReceiveRequest(context, out dividend, out divisior);
            }
            catch (Exception ex)
            {
                LogEntry entry = new LogEntry();
                entry.Title = Properties.Resources.BadRequestParameters;
                entry.Message = ex.Message;
                entry.Severity = TraceEventType.Error;
                entry.ExtendedProperties.Add("dividend", dividend);
                entry.ExtendedProperties.Add("divisior", divisior);
                Logger.Write(entry);

                this.RespondBadRequest(context);
                return;
            }

            SendResponse(context, dividend / divisior);
        }

        /// <summary>
        /// Odczytanie parametrów żądania klienta.
        /// </summary>
        /// <param name="context">Kontekst żądania.</param>
        /// <param name="dividend">Liczba dzielona.</param>
        /// <param name="divisior">Liczba dzieląca.</param>
        private void ReceiveRequest(HttpContext context, out int dividend,
            out int divisior)
        {
            dividend = Convert.ToInt32(context.Request.QueryString["a"], null);
            divisior = Convert.ToInt32(context.Request.QueryString["b"], null);

            if (divisior == 0)
                throw new ArgumentOutOfRangeException("divisior");
        }

        /// <summary>
        /// Wysłanie odpowiedzi do klienta.
        /// </summary>
        /// <param name="context">Kontekst żądania.</param>
        /// <param name="result">Wynik dzielenia.</param>
        private void SendResponse(HttpContext context, int result)
        {
            string message = string.Format(Properties.Resources.DevisionResult,
                result);

            context.Response.ContentType = "text/plain";
            context.Response.Write(message);

            Logger.Write(message);
        }
```

Procedura `ProcessRequest` przetwarza żądanie klienta: odczytuje w metodzie `ReceiveRequest` przekazane w zapytaniu parametry, wykonuje obliczenia a następnie korzystając z `SendResponse` wysyła odpowiedź. Jak widzimy początek `ProcessRequest` to zarejestrowanie informacji w logu o fakcie żądania dzielenia dwóch liczb całkowitych. Następnie ma miejsce odczytanie parametrów zapytania. Jakikolwiek błąd skutkuje jego rejestracją i zakończeniem obsługi żądania. Warto zwrócić uwagę, iż w przypadku błędu próbujemy również zarejestrować odczytane parametry żądania (stąd zresztą wynika konieczność inicjacji zmiennych dividend oraz divisior mimo, iż `ReceiveRequest` zwraca parametry poprzez out). Poprawne wykonanie operacji dzielenia powoduje wysłanie odpowiedzi oraz zarejestrowanie wyniku.

Jak już wcześniej zostało powiedziane, istnieje możliwość definiowania również własnych kategorii zdarzeń. Prezentuje to poniższy kod:

```csharp
        /// <summary>
        /// Metoda wywoływana w momencie zajścia w aplikacji nie obsłużonego wyjątku.
        /// </summary>
        /// <param name="sender">Obiekt sygnalizujący zdarzenie.</param>
        /// <param name="e">Parametry zdarzenia.</param>
        protected void Application_Error(object sender, EventArgs e)
        {
            Exception exception = this.Server.GetLastError();

            LogEntry entry = new LogEntry();
            entry.Categories.Add("Global");
            entry.Title = Properties.Resources.UnhandledException;
            entry.Message = exception.Message;
            entry.Severity = TraceEventType.Critical;
            Logger.Write(entry);
        }
```

W tym przypadku kategoria ta nazywa się **Global** (choćby dlatego, iż kod pochodzi z pliku **Global.asax**).

Pozostaje nam jeszcze skonfigurować bibliotekę rejestrującą zdarzenia. W przeciwnym bowiem razie aplikacja zgłosi nam wyjątek związany z niemożnością znalezienia właściwej sekcji konfiguracyjnej. Aby to zrobić klikamy prawym klawiszem na plik konfiguracyjny **Web.config** a następnie wybieramy polecenie **Edit Enterprise Library Configuration**. W oknie, które nam się pokaże wybieramy ponownie prawym klawiszem plik konfiguracyjny (tym razem podawany jest wraz z pełną ścieżką dostępu), a następnie klikamy **New -> Logging Application Block**. Dzięki temu w pliku konfiguracyjnym zostaną utworzone odpowiednie wpisy umożliwiające uruchomienie aplikacji. Domyślne ustawienia powodują rejestrację zdarzeń w dzienniku aplikacji (application event log).

{{< figure src="images/LAB.png" title="Konfiguracja Logging Application Block" >}}

Dodamy jeszcze możliwość rejestracji zdarzeń w pliku tekstowym. W tym celu klikamy prawym klawiszem myszy gałąź **Trace Listeners** a następnie **New -> Flat File Trace Listener**. Samo dodanie elementu odbierającego zdarzenia to jeszcze za mało. Należy jeszcze dodać w gałęzi **Category Sources / General** odwołanie do dodanego przed chwilą ujścia plikowego (**New -> Trace Listener Reference**).

Ponieważ w naszej przykładowej aplikacji skorzystaliśmy również z możliwości definiowania własnych kategorii, gałąź **Category Sources** uzupełnimy o właściwy wpis. W tym celu po wybraniu tej gałęzi prawym przyciskiem wybierzemy polecenie **New -> Category**, zmienimy nazwę na nowej kategorii na **Global** (zgodnie z przykładowym kodem pliku **Global.asax**). Następnie dodamy do nowej kategorii odwołanie do pliku (**New -> Trace Listener Reference**). Dzięki temu, zdarzenia opisane kategorią **Global** będą rejestrowane jedynie w pliku tekstowym.

Tak przygotowana i skonfigurowana aplikacja jest już gotowa do pracy i zapisywania zdarzeń w dzienniku aplikacji (application event log) oraz w pliku tekstowym.

## Zadanie konkursowe #1 - Logging Application Block, Enterprise Library 3.1

Zgodnie z obietnicą i założeniami konkursu na koniec zadania do wykonania:

1. Zrefaktoryzować kod aplikacji konkursowej tak, aby rejestrowane zdarzenia można było grupować według warstwy prezentacji, logiki i danych (kategorie: UI, Model, DataServices);
2. Logi dla warstwy prezentacji powinny być rejestrowane w jednym ujściu, a zdarzenia płynące z modelu i danych w drugim;
3. Plik konfiguracyjny **.config** nie może zawierać logowania do dziennika zdarzeń (event log).

Uwagi

1. Zalecane ujście dla zdarzeń to pliki tekstowe;
2. Jeśli w aplikacji konkursowej brak jest logów to należy je dodać. W przypadku modelu wystarczą informacje o fakcie wywołania metod wraz z wartościami zmiennych, na podstawie których podejmowane są decyzje. Dla warstwy danych wystarczy tylko powiadomienie o fakcie wywołania metody.

Rozwiązane zadania należy przesyłać na adres mgrzeg at gmail kropka com. Reguły wiadomości:

1. Temat wiadomości: [ENTLIB LAB] Rozwiązanie zadania konkursowego;
2. Treść (może być w załączonym pliku .txt, .doc, .rtf) powinna zawierać krótki opis co zostało wykonane;
3. Plik .zip zawierający kod  aplikacji konkursowej po refaktoryzacji. Można dołączyć również kod wykonywalny (bez bibliotek EntLib, etc. - tylko to, co niezbędne);
4. Dane adresowe do wysyłki nagrody - NIE! Skontaktujemy się ze "szczęśliwcem" via email i ustalimy szczegóły.

Adresy:

* [Kod aplikacji przykładowej](http://zine.net.pl/files/folders/entlib/entry458.aspx) opisywanej w powyższym tekście;
* [Kod aplikacji konkursowej](http://zine.net.pl/files/folders/entlib/entry459.aspx), którą należy poddać refaktoryzacji.

W razie wątpliwości etc. proszę zadawać pytania.