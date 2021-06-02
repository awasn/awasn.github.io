---
title: "HTTP - Własny serwer komunikacji. Część #1 - Wstęp"
date: 2008-01-14T22:45:00+01:00
languageCode: pl-pl
description: Implementacja w pełni funkcjonalnego serwera komunikacji korzystającego z protokołu HTTP, opartego o śodowisko ASP.NET oraz IIS
categories:
  - Development
tags:
  - .NET
  - .NET Compact Framework
  - ASP.NET
  - IIS
  - Mobile
series:
  - Serwer komunikacji
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/01/14/http-w-asny-serwer-komunikacji-cz-1.aspx
disqus_disable: true
---

Budując rozwiązanie, które wymaga wymiany informacji z aplikacjami uruchomionymi na innych maszynach możemy skorzystać z kilku propozycji, jakich dostarcza platforma .NET. Są to między innymi gniazdka (sockets), .NET Remoting, Usługi Sieciowe (Web Services), kolejki wiadomości (MSMQ) czy łączący i uzupełniający większość z tych rozwiązań Windows Communication Foundation (WCF). Oczywiście część z tych metod działa praktycznie tylko wtedy, jeśli obie strony oparte są o .NET (.NET Remoting czy net.tcp z WCF). Inne znów mają problemy jeśli wychodzimy poza obręb danej sieci (.NET Remoting). Nawet jednak jeśli obie ze stron oparte są o platformę .NET to również mogą wystąpić kłopoty. Weźmy urządzenia mobilne, które korzystają z .NET Compact Framework. Platforma .NET CF jest podzbiorem możliwości .NET. Nie zawiera .NET Remoting, nie obsługuje serializacji binarnych etc. Oczywiście, jeśli naszym zamiarem jest przesłanie niewielkich ilości danych możemy skorzystać z Usług Sieciowych. Jest też protokół FTP czy SMTP. Nie sposób wszystkich możliwych rozwiązań wymienić bo i wiedzy zwyczajnie brak.

Podjęcie decyzji w kwestii wyboru sposobu wymiany danych nie jest wbrew pozorom trywialne. Na początek dobrze zawsze określić swoje potrzeby i wymagania. A jakie są nasze zamiary? Załóżmy, iż po stronie serwera interesuje nas rozwiązanie, które spełni następujące warunki:

* Pozwoli przesłać do kilku megabajtów danych;
* Będzie istnieć możliwość wywoływania na żądanie klienta określonych akcji;
* Dostęp do danych dozwolony po podaniu właściwej nazwy użytkownika i hasła;
* Przesyłane informacje będą zabezpieczone;
* Dostęp do zasobów możliwy spoza własnej sieci (intranetu);
* Łatwość konfiguracji i zarządzania.

Dodatkowo aplikacje klienckie:

* Są uruchamiane na komputerach stacjonarnych oraz na urządzeniach mobilnych;
* Nie są oparte tylko o .NET. Dopuszczamy możliwość korzystania z .NET Compact Framework, C++ wraz z MFC itp.;
* Mogą być uzupełniane o nowe formy komunikacji.

Konieczność działania rozwiązania na wielu platformach i środowiskach oznacza, iż na placu boju pozostają gniazdka oraz Usługi Sieciowe oraz rozwiązania oparte o FTP.

Rozprawmy się wpierw z drugim rozwiązaniem. Usługi sieciowe oparte są o XML. Powoduje to bardzo dużą nadmiarowość ilości przesyłanych danych. W ramach środowiska ASP.NET istnieje również ograniczenie co do ilości przesyłanych danych. Standardowo jest ono ustalone na 4 MB, aczkolwiek można je zmienić. Wprowadzenie kompresji na poziomie serwera IIS nic nie pomoże, ponieważ pakowanie jest wykonywane dopiero po załadowaniu przez ASP.NET danych przed wysłaniem do pamięci. Oznacza to, iż faktycznie danych użytecznych w pakiecie HTTP będzie, w zależności od budowy struktury XML, nawet kilkadziesiąt razy mniej niż wynosi sam rozmiar pakietu! Poza tym przesyłanie danych w postaci tekstu powoduje po stronie słabszego w zasoby klienta, a takimi są np. urządzenia oparte o Windows Mobile, duży czas oczekiwania na przetworzenie odebranych informacji. Można oczywiście przesłać dane w postaci binarnej jako strumień bajtów w oparciu o własne mechanizmy zapisu typów. Aby taka operacja była możliwa w ramach Usług Sieciowych strumień ten musi być jednak przetworzony przy pomocy protokołu [Base64](http://pl.wikipedia.org/wiki/Base64), co powoduje, iż rozmiar przesyłanych informacji wzrasta o 33%.

File Transfer Protocol. No cóż. Mało jest implementacji klientów spełniających wymagania dotyczące bezpieczeństwa i działających na dowolnych maszynach. Tak naprawdę to nie wiem czy w ogóle takie wszechstronne implementacje istnieją. Poza tym nie ma możliwości w przeważającej większości serwerów wywoływania określonych akcji na żądanie klienta. Do tego protokół FTP ma jedną zasadniczą wadę: jest nieprzyjazny dla administratora. Klient standardowo łączy się z serwerem na porcie 21, loguje się a następnie wydaje komendy, których większość skutkuje przesłaniem strumienia danych. Dla każdego przesłania danych tworzone jest osobne połączenie. O numerze portu, na którym dany kanał do wymiany danych powstanie, decyduje się na dwa sposoby. W trybie aktywnym (ACTIVE) klient podaje adres i numer portu, z którym serwer powinien się połączyć. Z powodu NAT czyli translacji adresów metoda ta nie działa jeśli jesteśmy poza własną siecią firmową i nie mamy stałego adresu IP. Wówczas stosuje się tryb pasywny (PASSIVE), gdzie to serwer podaje numer portu, z którym klient powinien się połączyć. Oznacza to, iż reguła na ścianie ogniowej w trybie pasywnym serwera FTP musi zezwalać na wszystkie połączenia przychodzące! Oczywiście niektóre serwery umożliwiają wybór zakresu portów dostępnych dla klientów, nie mniej jednak rozwiązanie takie pozostawia wiele do życzenia. Otwarte porty na komputerze dostępnym publicznie, ze stałym adresem IP, to gwarantowane kłopoty.

A tak na marginesie. Być może niektórzy przypomną sobie teraz problemy programu Internet Explorer z obsługą protokołu FTP i dodanie do konfiguracji opcji przełączającej przeglądarkę z trybu aktywnego na pasywny.

Cóż wydaje się więc, iż jeśli chcemy skorzystać z wydajnego przesyłania danych pozostało nam napisanie od A do Z własnego rozwiązania wykorzystującego gniazdka. Nie jest to trudne. Silnik takiego serwera spełniający wcześniejsze założenia i pracujący w trybie asynchronicznym (wydajność!) nie powinien przekroczyć 5 000 linii kodu w języku C#. Klient takiego rozwiązania to maksimum 2 000 linii kodu. Problem pojawia się gdzie indziej. Jak ma wyglądać zarządzanie takim serwerem? Co z dostępem zdalnym? Co z kontrolą zawieszonych połączeń i wątków? Tutaj może być mało i 25 000 linii czystego kodu. No i jeszcze testy.

Okazuje się jednak, iż środowisko Windows wraz z platformą .NET dostarczają nam bardzo wydajnej infrastruktury umożliwiającej tworzenie własnych rozwiązań do wymiany danych, których protokół oparty jest o HTTP. Internet Information Services wraz z ASP.NET i sterownikiem HTTP.SYS stanowią idealną bazę dla wszelkiego tego typu projektów. Jednocześnie zapewniają nam takie elementy jak zdalny dostęp przez WWW czy automatyczne restartowanie aplikacji. Protokół HTTP jest przy tym zrozumiały dla zapór ogniowych sprzętowych i programowych. Konfiguracja reguł dla serwera jest banalna i co najważniejsze nie wymaga zezwolenia na połączenia przychodząca na dowolnym porcie. Wystarczy tylko jeden port.

W dalszych częściach postaram się przedstawić w jaki sposób zbudować w pełni funkcjonalny serwer komunikacji korzystający z protokołu HTTP, oparty o śodowisko ASP.NET oraz IIS. Kolejne odcinki będą poruszały takie tematy jak:

* Przygotowanie środowiska programistycznego oraz wybór narzędzi;
* Opis środowiska uruchomieniowego ASP.NET w kontekście protokołu HTTP;
* Przedstawienie szczegółowych założeń co do tworzonej aplikacji;
* Identyfikacja użytkowników i szyfrowanie przesyłanych informacji;
* Obsługa żądań HTTP;
* Synchroniczna obsługa żądań HTTP;
* Asynchroniczna obsługa żądań HTTP;
* Obsługa żądań HTTP w osobnym wątku;
* Wywoływanie bibliotek zewnętrznych;
* Konfiguracja aplikacji;
* Konfiguracja środowiska uruchomieniowego;
* Bezpieczeństwo rozwiązania.

Zapraszam.