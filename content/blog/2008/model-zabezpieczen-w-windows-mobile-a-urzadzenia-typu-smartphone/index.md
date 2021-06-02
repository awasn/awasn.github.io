---
title: Model zabezpieczeń w Windows Mobile a urządzenia typu Smartphone
date: 2008-02-27T16:53:00+01:00
lastmod: 2018-09-17T13:35:00+02:00
languageCode: pl-pl
description: Opis modelu zabezpieczeń security model w Windows Mobile. Konfiguracja urządzenia typu Smartphone
categories:
  - Development
tags:
  - Mobile
  - Visual Studio
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/02/27/model-zabezpiecze-w-windows-mobile-a-urz-dzenia-typu-smartphone.aspx
disqus_disable: true
---

Wraz z systemami Windows Mobile 5.0 i Windows Mobile 6 wprowadzono nowych model zabezpieczeń (security model), który umożliwia przy pomocy zasad (policies), ról (roles) oraz certyfikatów (certificates) kontrolowanie konfiguracji, zdalnego dostępu oraz wykonywanie aplikacji na urządzeniu mobilnym. Ograniczeniu podlega dostęp do zasobów takich jak przestrzeń dyskowa, rejestr systemu czy API. Na podstawie ustalonego poziomu zabezpieczeń, program może być uruchomiony z uprawnieniami:

* Uprzywilejowany - możliwość wywoływania API bez ograniczeń, dokonywania modyfikacji rejestru, pełny dostęp do systemu plików;
* Normalny - brak możliwości wywoływania API newralgicznych dla właściwego działania systemu, brak możliwości modyfikacji niektórych gałęzi rejestru, ograniczenia w dostępie do systemu plików. Na tym poziomie nie ma również możliwości instalowania certyfikatów;
* Blokowany - brak dostępu do jakichkolwiek zasobów systemu.

Poziom zabezpieczeń ustalany jest przez producenta lub dostawcę urządzenia. Mówimy o dostępie w modelu:

* Jednowarstwowym - sprawdzane jest czy uruchamiana aplikacja jest podpisana cyfrowo. Jeśli certyfikat jest poprawny (znajduje się w magazynie certyfikatów urządzenia) program uruchamiany jest z uprawnieniami uprzywilejowanymi. W przypadku braku podpisu lub podpisaniu przez certyfikat, który nie może być zweryfikowany, system wyświetla użytkownikowi okno dialogowe z pytaniem o zezwolenie na uruchomienie oprogramowania. Akceptacja oznacza uruchomienie aplikacji z uprawnieniami uprzywilejowanymi;
* Dwuwarstwowym - w przypadku oprogramowania podpisanego certyfikatem poprawnym i weryfikowalnym, poziom uprawnień, z którymi zostanie aplikacja uruchomiona, zależy od rodzaju certyfikatu. Certyfikaty składowane w magazynie certyfikatów uprzywilejowanych uprawniają do uruchomienia programu z uprawnieniami uprzywilejowanymi. Pozostałe certyfikaty dają możliwość uruchomienia aplikacji z uprawnieniami normalnymi. W przypadku braku podpisu lub podpisaniu przez certyfikat, który nie może być zweryfikowany, program może być uruchomiony co najwyżej z uprawnieniami normalnymi.

Istnieje oczywiście możliwość takiej konfiguracji urządzenia, aby pewne czynności były wykonywane przez system automatycznie np. bez wymagania potwierdzania przez użytkownika. Ważne też jest ustawienie przez producenta lub dostawcę urządzenia roli, kto może modyfikować poziom zabezpieczeń!

Co ciekawe model dwuwarstwowy dostępny jest jedynie w Windows Mobile przeznaczonym dla telefonów komórkowych (Windows Mobile 5.0 for Smartphone oraz Windows Mobile 6 Standard). I na dodatek jest to zalecany model domyślny.

Z punktu widzenia producenta lub dostawcy urządzenia ustawienie poziomu zabezpieczeń na dwuwarstwowy, z blokadą zmiany przez użytkownika właściwości tego poziomu, ogranicza do minimum problemy związane z wirusami, trojanami etc. Dla niewymagającego użytkownika ograniczenia te nie mają właściwie znaczenia. Osoba pragnąca wzbogacić urządzenie o dodatkowe oprogramowanie, przy pierwszym uruchomieniu aplikacji będzie zmuszona jedynie tę operację zaakceptować.

{{< figure src="images/SmartphoneSecurity_Default.png" title="Domyślne ustawienia i poziom zabezpieczeń urządzania typu smartphone" >}}

Programista natomiast staje niestety przed poważnym problemem. Visual Studio bowiem nie jest wstanie korzystać ze wszystkich zasobów urządzenia mobilnego, które są niezbędne do przeprowadzenia operacji zdalnego dostępu. Nie istnieje również, o czym było wcześniej, możliwość wgrania własnych certyfikatów!

Jeśli mamy zainstalowane Windows Mobile SDK w katalogu **\<Miejsce instalacji SDK\>\Windows Mobile 6 SDK\Tools\Security\Security Powertoy** znajdziemy program **SecCfgMgr.msi**, który wdroży na komputerze stacjonarnym aplikację **Security Configuration Manager** umożliwiającą sprawdzanie i modyfikacje ustawień poziomu zabezpieczeń komputerów mobilnych (w naszym przypadku telefonów). Oczywiście jeśli konfiguracja urządzenia na te modyfikacje będzie zezwalać. A co w przypadku, jeśli uprawnienia do zmiany ustawień poziomu zabezpieczeń będą przypisane do dostawcy urządzenia tak jak to widzimy na rysunku powyżej (pole **Grant manager role to**)? Musimy mieć nadzieję, iż istnieje szansa programowej zmiany konfiguracji z wykorzystaniem narzędzi (aplikacji) podpisanych uprzywilejowanymi certyfikatami.

Pierwszym dostawcą telefonów komórkowych wyposażonych w system Windows Mobile (Windows Mobile 5.0 for Smartphone oraz Windows Mobile 6 Standard), który wprowadził blokady na poziomie dwuwarstwowym był Orange. Na szczęście dla urządzeń sieci Orange dostępna jest strona `http://spvunlock.rd.francetelecom.com`, która po podaniu numeru EMEI urządzenia, numeru telefonu, imienia i nazwisko (prowdopodobnie weryfikacja czy telefon wykorzystywany jest w sieci Orange) oraz adresu email wysyła do nas plik **remove_\<numer EMEI\>**.cab umożliwiający zdjęcie zabezpieczeń z urządzenia (unlock). Po usunięciu blokady urządzenie powinno oczywiście również szybciej pracować. System ma bowiem mniejszy zakres danych do weryfikacji.

{{< figure src="images/SmartphoneSecurity_AfterUnlock.png" title="Urządzenie typu smartphone po zdjęciu blokady zabezpieczeń" >}}

Jak widzimy na powyższym rysunku, po zdjęciu zabezpieczeń telefon został przestawiony na jednowarstwowy poziom zabezpieczeń, skonfigurowany dodatkowo tak, iż dozwolone są wszystkie operacje bez informowania o tym użytkownika. Oczywiście teraz można i należy trochę podwyższyć poziom zabezpieczeń, chociażby poprzez wymóg pytania się przez system czy na pewno chcemy uruchomić niepodpisaną ważnym certyfikatem aplikację.

Konfiguracja poziomu zabezpieczeń urządzenia typu smartphone przyjazna dla programisty

{{< figure src="images/SmartphoneSecurity_Manager.png" title="Konfiguracja poziomu zabezpieczeń urządzenia typu smartphone przyjazna dla programisty" >}}

Teraz możemy przystąpić do programowania.

Na koniec możemy również uzupełnić jeszcze urządzenie mobilne o kilka certyfikatów. W katalogu **\<Miejsce instalacji SDK\>\Windows Mobile 6 SDK\Tools\Security\SDK Development Certificates** znajduje się plik **Certs.cab**, który po wgraniu i uruchomieniu na urządzeniu mobilnym instaluje certyfikaty umożliwiające bez przeszkód korzystanie z narzędzi dostępnych w ramach SDK. Zgodnie z tym co było powiedziane wcześniej, przed zdjęciem blokady i zmianą ustawień poziomu zabezpieczeń instalacja ta zakończyłaby się niepowodzeniem.

Adresy i dodatkowe informacje:

* [All About "Application Locked"](https://blogs.msdn.microsoft.com/windowsmobile/2007/05/29/all-about-application-locked);
* Security Model for Windows Mobile 5.0 and Windows Mobile  (`http://www.microsoft.com/technet/solutionaccelerators/mobile/maintain/SecModel/aff7cf7f-0e11-4ef4-8626-f33bd969b35a.mspx?mfr=true`).