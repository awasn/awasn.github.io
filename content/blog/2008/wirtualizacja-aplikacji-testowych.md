---
title: Wirtualizacja aplikacji testowych
date: 2008-09-15T13:21:00+02:00
lastmod: 2018-09-17T13:29:00+02:00
languageCode: pl-pl
description: Wirtualizacja aplikacji testowych z wykorzystaniem funkcjonalności dysków różnicowych
categories:
  - Tools
tags:
  - Virtualization
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/09/15/wirtualizacja-aplikacji-testowych.aspx
disqus_disable: true
---

Maszyn wirtualnych używam od dawna. I to korzystając zarówno z [VMware Server](https://www.vmware.com) jak i z [Virtual PC](https://www.microsoft.com/en-us/download/details.aspx?id=3702). A jednak po  przeczytaniu na `http://wss.pl` artykułu `Praca z programem Virtual PC 2007`, którego autorami są Robert Stuczynski i Jacek Doktór zorientowałem się, iż zupełnie nie wykorzystywałem do tej pory potęgi dysków różnicowych!

Z racji tworzonego oprogramowania niejednokrotnie wykonuję integrację oprogramowania mobilnego z zewnętrznymi systemami np. sprzedaży. Tego typu systemy są dostępne do testów często w wersjach z ograniczeniem czasowym. Zdarza się, iż interesujący mnie komponent działa tylko kilkanaście dni. Dłużej trwa oczekiwanie na odpowiedź klienta w sprawie oferty. Ile razy można przeinstalowywać środowisko programisty?

Korzystając z funkcjonalności dysków różnicowych utworzyłem dysk bazowy zawierający system operacyjny, Visual Studio oraz kilka przydatnych narzędzi. Teraz pozostaje zainstalować na dysku różnicowym tylko system testowy. W przypadku upłynięcia daty ważności licencji konieczne będzie jedynie utworzenie nowego dysku różnicowego.