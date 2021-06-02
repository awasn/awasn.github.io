---
title: Z Virtual PC do VMware Server
date: 2007-09-04T22:47:00+02:00
languageCode: pl-pl
description: Rozwiązanie problemu importu maszyny wirtualnej z Virtual PC do VMware Server przy włączonej opcji zapisywania modyfikacji poza wirtualnym dyskiem
categories:
  - Tools
tags:
  - Virtualization
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/09/04/z-virtual-pc-do-vmware-server.aspx
disqus_disable: true
---

Zgodnie z deklaracjami firmy VMware, narzędzia tego producenta służące do wirtualizacji komputerów i systemów bez problemu obsługują format stosowany przez aplikacje Virtual PC oraz Virtual Server firmy Microsoft. Wielkie było moje zaskoczenie kiedy okazało się, iż jednak nie do końca. Otóż nie powiedzie nam się operacja uruchomienia (czy to poprzez polecenie **Open** czy też **Import** w przypadku VMware Server) maszyny wirtualnej w formacie Microsoft, która skonfigurowana jest tak, aby używała opcji **Undo disks**, czyli zapisywania wszelkich modyfikacji poza wirtualnym dyskiem.

{{< figure src="images/VMware_Server_Import.png" title="Nieudany import maszyny wirtualnej do VMware Server" >}}

Modyfikacje te przechowywane są w pliku o rozszerzeniu .vsv. Na szczęście rozwiązanie problemu jest trywialne. Chcąc skorzystać z programów VMware plik ten należy wpierw po prostu skasować.