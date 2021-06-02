---
title: Loopa TV
date: 2007-09-14T13:43:00+02:00
languageCode: pl-pl
description: Instalacja wirtualnej karty sieciowej aby otrzymać adres IP w przypadku braku dostępu do sieci
categories:
  - Development
  - Tools
tags:
  - Mobile
  - Virtualization
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/09/14/loopa-tv.aspx
disqus_disable: true
---

Programując urządzenia mobilne człowiek posiada naturalną potrzebę uruchomienia kodu na takim komputerze (komputerku?). W ramach standardowego środowiska programisty nie ma z tym zazwyczaj problemu. Bierzemy dok (vel kołyskę), wkładamy do niego urządzenie i łączymy z PC przy pomocy kabla USB. Działa. Jakaś komunikacja protokołem TCP z komputerem stacjonarnym? Nie ma problemu, działa. Robimy to samo w terenie, albo bez sieci (nie mając połączenia)... no... nie działa. Nie da rady skomunikować się poprzez TCP z urządzenia mobilnego. Ki diabeł? A tak właściwie jak nie ma sieci, to jaki mamy adres IP? I jak urządzenie mobilne odnajdzie naszego peceta czy też notebooka?

Problem nie trywialny, ale na szczęście prosty jeśli chodzi o rozwiązanie. Wszystko co nam potrzeba w takich sytuacjach to instalacja Microsoft Loopback Adapter, czyli wirtualnej karty sieciowej. Dzięki temu nasz komputer otrzyma automatycznie adres z zakresu 169.254.xxx.xxx i życie stanie się prostsze.

Co ciekawe powyższe rozwiązanie sprawdza się również w przypadkach, kiedy mimo posiadania połączenia sieciowego również nie jesteśmy wstanie wykonać transmisji z urządzenia mobilnego. Czasami bowiem powstają w systemie stacjonarnym konflikty z istniejącą już kartą sieciową lub jej sterownikami. Inny zakres zastosowań to budowa sieci przy korzystaniu z oprogramowania do wirtualizacji.

Adresy i dodatkowe informacje:

* `http://support.microsoft.com/kb/236869` - Instalacja karty Loopback w systemie Windows 2000;
* `http://support.microsoft.com/kb/839013` - Instalacja karty Loopback w systemie Windows XP.