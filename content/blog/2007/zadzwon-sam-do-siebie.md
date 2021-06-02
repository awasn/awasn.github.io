---
title: Zadzwoń sam do siebie
date: 2007-07-04T22:23:00+02:00
languageCode: pl-pl
description: Wykorzystanie komend AT do sterowania telefonem komórkowym
categories:
  - Tools
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/07/04/zadzwo-sam-do-siebie.aspx
disqus_disable: true
---

Dennis Hayes pod koniec lat siedemdziesiątych wraz z modemem *Hayes 300 baud Smartmodem* zaproponował zestaw poleceń sterujących tymi urządzeniami. Polecenia te nazywamy dzisiaj komendami AT i są one obsługiwane między innymi przez telefony komórkowe. Jeśli mamy możliwość podłączenia komórki do komputera - czy to przez dongla Bluetooth, czy też korzystając z kabla USB - możemy spróbować zdalnie pozarządzać telefonem.

Wpierw jednak mała uwaga. Zarządzanie modemem odbywa się poprzez standard RS-232. Dlatego też Bluetooth, z którego zamierzamy skorzystać powinien obsługiwać profil portu szeregowego. Ponieważ każdy producent Bluetooth czy telefonu komórkowego posiada własne oprogramowanie, czeka nas pewien wysiłek związany z instalacją i konfiguracją sterowników jeśli dane urządzenie tego wymaga. Osobiście do tej pory łączyłem następujące urządzenia:

* Siemens S65 poprzez Bluetooth z komputerem PC, który do portu USB miał włożonego odpowiedniego dongla. Bluetooth w PC wymagał zainstalowania sterowników i podania portu szeregowego, który miał być z donglem skojarzony;
* Sony Ericsson K750i poprzez kabel USB z notebookiem. W komputerze miałem zainstalowane oprogramowanie producenta do zarządzania telefonem komórkowym.

Generalnie rzecz biorąc Menedżer urządzeń powinien pokazywać nam jeden z portów szeregowych przypisany do urządzenia, z którym będziemy się komunikować.

Do wydawania poleceń wykorzystać możemy stary, poczciwy HyperTerminal. Tworzymy w nim nowe połączenie, dla którego wybieramy właściwy port. I możemy zaczynać. Najprostszą metodą umożliwiającą sprawdzenie czy połączenie zostało faktycznie nawiązane jest wpisanie w terminalu komendy (pamiętajmy, iż zatwierdzenie polecenia następuję po wciśnięciu klawisza Enter):

```dos
AT
```

Telefon powinien zwrócić nam odpowiedź OK. Jeśli ktoś już się pobudził i marzy o zalewaniu bliźnich wiadomościami SMS to pragnę ostudzić zapał. Kodowanie SMS do formatu wymaganego przez telefon komórkowy jest nie do wykonania w pamięci operacyjnej człowieka. Trzeba skorzystać z dodatkowych narzędzi. Warto również poszukać w sieci zestawu komend AT dla posiadanego modelu telefonu. Oczywiście istnieje duża część wspólna zwana standardem implementowana przez każde urządzenie.

Na koniec jeszcze jeden przykład. Poniższa sekwencja pokazuje nawiązanie i zerwanie połączenia:

```dos
ATD555123456;
OK
AT+CHUP
OK
```

Korzystając z kontrolki `SerialPort` w ramach .NET możemy za pomocą powyższej kombinacji... A swoją drogą, czy ktoś wie, iż można zadzwonić do samego siebie?