---
title: "HTTP - Własny serwer komunikacji. Część #2 - Środowisko programistyczne"
date: 2008-01-27T22:12:00+01:00
lastmod: 2018-09-17T13:42:00+02:00
languageCode: pl-pl
description: Implementacja w pełni funkcjonalnego serwera komunikacji korzystającego z protokołu HTTP, opartego o śodowisko ASP.NET oraz IIS
categories:
  - Development
  - Tools
tags:
  - .NET
  - ASP.NET
  - IIS
  - Virtualization
series:
  - Serwer komunikacji
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/01/27/http-w-asny-serwer-komunikacji-cz-2-rodowisko-programistyczne.aspx
disqus_disable: true
---

Zasadniczo całe rozwiązanie zbudowane będzie przy pomocy Microsoft Visual Studio 2005 Professional Edition w oparciu o ASP.NET Web Application jako typ projektu. Użytkownicy wersji Express będą niestety zmuszeni dokonać konwersji do projektu Web Site. Dostarczany z Visual Studio serwer WWW w zupełności wystarczy do uruchomienia prezentowanego rozwiązania. Z racji wybranego środowiska operować będziemy na platformie .NET w wersji 2.0. Nie mniej jednak bez żadnych zmian prezentowane oprogramowanie będzie działało również w wersji 1.1 oraz 3.x platformy .NET. W kolejnych częściach nie będziemy się też wgłębiać w tajniki systemu Windows Server 2008 oraz IIS 7 dostępnych obecnie w wersji Release Candidate. Naszą bazę "dydaktyczną" będzie stanowił Windows Server 2003 R2.

## Biblioteki

### NLog

Na różnych etapach tworzenia oprogramowania oraz w czasie produkcyjnego wykorzystywania rozwiązania istnieje konieczność rejestrowania (logowania) działania aplikacji. Można do tego  celu wykorzystać standardowe możliwości platformy .NET. Można w tym celu skorzystać z [Enterprise Library](http://msdn.microsoft.com/entlib), a konkretnie z części nazywanej [Logging Application Block](http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/09/24/konkurs-enterprise-library-logging-lab.aspx). W budowanym rozwiązaniu będziemy wykorzystywać jeszcze inne rozwiązanie, mianowicie napisaną przez Jarka Kowalskiego bibliotekę [NLog](http://nlog-project.org). Na stronach projektu można zapoznać się z napisanym po polsku wprowadzeniem (`http://www.nlog-project.org/wprowadzenie.html`) do korzystania z biblioteki. Polecam. Tym bardziej, że NLog może też być wykorzystywany na platformie .NET Compact Framework.

## Narzędzia

### WFetch

WFetch to jedno z ciekawszych narzędzi. Zezwala na wszechstronną diagnostykę aplikacji Web poprzez umożliwienie użytkownikowi wysyłanie i oglądanie na poziomie przesyłanych pakietów HTTP komunikacji pomiędzy programem a serwerem WWW. Program można pobrać między innymi ze strony <https://support.microsoft.com/en-us/help/284285/how-to-use-wfetch-exe-to-troubleshoot-http-connections>. Poniżej przykładowy efekt działania:

{{< figure src="images/WFetch.png" title="WFetch - żądanie i odpowiedź zwrotna" >}}

Jak widzimy do aplikacji Web uruchomionej lokalnie na porcie 1395 (serwer testowy Visual Studio) wysyłamy żądanie wyświetlenia domyślnej strony. Reakcja serwera jest zgodna z naszymi oczekiwaniami. Dostajemy bowiem jako odpowiedź zawartość właściwego pliku.

### Network Monitor

W czasie tworzenia rozwiązań przesyłających dane poprzez sieć dobrze jest mieć możliwość sprawdzenia poprawności wykonanej implementacji, czy informacje są dobrze zabezpieczone etc. W tym celu można wykorzystać narzędzie [Microsoft Network Monitor](https://www.microsoft.com/en-us/download/details.aspx?id=4865), które umożliwia obserwowanie (przechwytywanie) oraz analizowanie ruchu sieciowego. Informacje dotyczące konfiguracji oraz użytkowania tej aplikacji można znaleźć na blogu [Network Monitor](http://blogs.technet.com/netmon) oraz oczywiście w pomocy programu.

Po zainstalowaniu i uruchomieniu programu po lewej stronie, w części **Capture Network Traffic**, klikamy przycisk **Create a new capture tab...** dzięki czemu będziemy mogli rozpocząć przechwytywanie ruchu sieciowego. Aby nie zostać zalanym informacjami, w utworzonej zakładce, w sekcji **Display Filter** wpisujemy **protocol.HTTP** i klikamy **Apply**. Pozwoli nam to ograniczyć wyświetlane dane tylko do protokołu HTTP. Klawisz F10 uruchamia nasłuch, F11 zatrzymuje.

{{< figure src="images/Network_Monitor_Request.png" title="Network Monitor - Żądanie wyświetlenia domyślnej strony" >}}

Powyższy zrzut ekranu pokazuje wysłane do serwera żądanie wyświetlenie domyślnej strony dla aplikacji Sample1.

## Wirtualizacja

W przypadku tworzenia rozwiązań sieciowych pewnym problemem może być testowania rozwiązania. Chodzi mi tutaj przede wszystkich o kwestie przesyłania danych, poprawności implementacji czy bezpieczeństwo. Ruch sieciowy najłatwiej jest analizować pomiędzy dwoma komputerami. Czasem w możliwościach ogranicza nas system operacyjny. Np. Windows XP Home Edition nie posiada serwera IIS. Rozwiązaniem tych problemów, i nie tylko tych, jest wirtualizacja czyli skorzystanie z preinstalowanych systemów umożliwiających nam w zdefiniowanym okresie czasu między innymi testowanie i sprawdzanie poprawności działania aplikacji Web. Z wirtualizacją wiążą się dwa pojęcia. Komputer na którym uruchamiamy oprogramowanie wirtualizacyjne nazywamy hostem. System uruchamiany zaś w ramach tego oprogramowania to gość.

Aby móc skorzystać z możliwości maszyn wirtualnych musimy mieć zainstalowane oprogramowanie ową wirtualizację umożliwiające. Polecam dwa. Pierwsze z nich to [VMware Server](https://vmware.com), drugie zaś to [Virtual PC](https://www.microsoft.com/en-us/download/details.aspx?id=3702). Oba rozwiązania są bezpłatne. Co prawda dostarczane przez Microsoft wirtualne dyski systemów zawierających Internet Information Services wymagają aplikacji Virtual Server (`http://www.microsoft.com/windowsserversystem/virtualserver`), ale jeśli ktoś ma Windows XP Home Edition to i tak tego programu nie uruchomi. Na szczęście zarówno VMware Server 1.0 jak i Virtual PC 2007 bez problemów umożliwiają uruchomienie tych maszyn wirtualnych a pojawiające się na początku z tego tytułu ostrzeżenia można spokojnie zignorować. Nie polecam obecnie instalowania VMware Server w wersji 2.0 Beta.

Aby pobrać maszynę wirtualną należy wejść na stronę `http://www.microsoft.com/vhd`. Z pośród dostępnych rozwiązań w chwili obecnej możemy wybrać między innymi [Windows Server 2003 R2](https://www.microsoft.com/en-us/download/details.aspx?id=19727) w wersji Enterprise. Pobrane pliki należy rozpakować do wybranej lokalizacji. Poniżej opis dodania, konfiguracji i uruchomienia tego systemu w ramach obu najważniejszych aplikacji umożliwiających wirtualizację.

### Dodawanie maszyny wirtualnej

Pierwszy krok na drodze do szczęścia to podłączenie maszyny wirtualnej do aplikacji wirtualizującej. W programie Virtual PC z menu **File** wybieramy polecenie **New Virtual Machine Wizard** i klikamy **Next**. Zaznaczamy opcję **Add an existing virtual machine**. W następnym oknie znajdujemy w wybranej lokalizacji plik **WS03R2EE.vmc** (jeśli pobraliśmy oczywiście Windows Server 2003 R2). Zakończenie kreatora umożliwi przejście do skonfigurowania dodanej maszyny wirtualnej. Na liście zaznaczamy **WS03R2EE** i klikamy przycisk **Settings**. W ustawieniach należy wziąć pod uwagę kilka parametrów:

* **Memory** - im więcej tym lepiej. To oczywiście zależy od ilości pamięci zainstalowanej na naszym fizycznym komputerze;
* **Networking** - dostęp do sieci. Domyślnie ustawiony jest brak połączenia. Opis konfiguracji tego parametru znajdziemy w dalszej części artykułu;
* **Shared Folder** - współdzielone między komputerem fizycznym a wirtualnym zasoby w postaci katalogów i plików. Opcja ta dostępna jest zazwyczaj dopiero po zainstalowaniu w systemie wirtualnym specjalnych dodatków integrujących. Ewaluacyjna wersja serwera te dodatki już posiada.

Przed uruchomieniem maszyny wirtualnej należy jeszcze w **File -> Options** w sekcji **Keyboard** kliknąć pole tekstowe **Current host key** a następnie wcisnąć na klawiaturze lewy klawisz Alt. W przeciwnym razie, przy domyślnie ustawionym klawiszu jakim jest prawy Alt nie będziemy mogli używać polskich znaków.

Pierwsze uruchomienie maszyny wirtualnej to automatyczna konfiguracja i restart. Po ponownym uruchomieniu możemy zalogować się do serwera. Pamiętajmy, iż zamiast Ctrl-Alt-Del używamy Alt-Del. Nazwa: Administrator. Hasło: Evaluation1.

W aplikacji VMware Server, podobnie jak w Virtual PC, musimy wpierw wirtualną maszynę dodać. Z menu **File** wybieramy polecenie **Import...** a następnie klikamy dwa razy przycisk **Next**. W oknie **Specify the Source Image** wybieramy plik konfiguracyjny maszyny wirtualnej (w naszym przypadku **WS03R2EE.vmc**) i klikamy **Next**. Import oznacza utworzenie kopii maszyny wirtualnej w formacie VMware - należy wziąć to pod uwagę i zadbać w o odpowiednią ilość wolnej przestrzeni na dysku.

{{< figure src="images/VMware_Server_Console.png" title="VMware Server - konsola zarządzająca" >}}

Jeśli import zakończy się klęską, należy sprawdzić, klikając łącze **Show the logfile**, czy w logu znajduje się informacja następującej treści: *TranslateError: original Description is Failed to initialize MSXML*. Tego typu komunikat oznacza, iż w systemie brakuje parsera XML w wersji 2.6, 3.0 lub 4.0. Stosowne wersje można pobrać ze stron firmy Microsoft. Np. wersję MSXML 4.0 Service Pack 2 można znaleźć [tutaj](https://www.microsoft.com/en-us/download/details.aspx?id=19662). Należy pobrać plik [msxml.msi](http://download.microsoft.com/download/9/6/5/9657c01e-107f-409c-baac-7d249561629c/msxml.msi) i zainstalować oprogramowanie.

Kolejny problem jaki może się pojawić to niemożność ustalenia przez importer VMware Server poprawnej ścieżki dostępu do wirtualnego  dysku **Win2k3R2EE.vhd**. Należy wówczas zmodyfikować plik **WS03R2EE.vmc** wyszukując w nim element

```xml
    <absolute type="string">Win2k3R2EE.vhd</absolute>
```

i wpisując pełną ścieżkę dostępu do wirtualnego dysku np.

```xml
    <absolute type="string">d:\Windows_Server_2003_RS\Win2k3R2EE.vhd</absolute>
```

W sumie zawsze mi się wydawało, iż bezwzględna ścieżka powinna zawierać literę dysku oraz pełną ścieżkę dostępu do danych zasobów. No ale...

Tak jak w przypadku Virtual PC powinniśmy przydzielić maszynie wirtualnej odpowiednią ilość pamięci. Jak mamy fantazję, to możemy z serwera zrobić nawet maszynę o dwóch procesorach.

Aby opuścić przestrzeń wirtualnego systemu należy nacisnąć kombinację Ctrl-Alt. Wysłanie kombinacji klawiszy Ctrl-Alt-Del można osiągnąć poprzez wybranie w menu VMware Server polecenia **VM -> Send Ctrl+Alt+Del**. Mając uruchomiony system powinniśmy w celu ułatwienia sobie życia zainstalować VMware Tools. Klikamy prawym klawiszem na zakładkę WS03R2EE lub na nazwę naszej maszyny na liście zainstalowanych systemów i wybieramy polecenie **Install VMware Tools...**. Dodatki te umożliwią nam między innymi korzystanie z maszyny wirtualnej tak jak z każdego innego programu, bez konieczności używania kombinacji Ctrl-Alt. Niestety brak jest współdzielonych folderów.

Po zainstalowaniu VMware Tools mogą wystąpić  problemy jeśli korzystamy z myszki podpiętej do portu USB. W takim wypadku należy dodatki odinstalować i zainstalować jeszcze raz wybierając **Custom** jako typ instalacji. Z listy **VMware Device Drivers** wykluczamy wówczas sterownik do myszki. Czasem również może zaistnieć potrzeba odrzucenia sterownika grafiki.

### Połączenia sieciowe

Po podłączeniu maszyny wirtualnej powinniśmy skonfigurować połączenia sieciowe. Ze względu na sposób w jaki będziemy chcieli wykorzystywać wirtualny system mamy do wyboru dwie opcje.

Pierwsza polega na podłączeniu systemu gościa bezpośrednio do karty fizycznej hosta. Maszyna wirtualna będzie wtedy widziana w sieci jako osobny komputer. Jednocześnie będziemy posiadać z niej pełny dostęp do Internetu. W programie Virtual PC wybieramy polecenie **Settings** dla danej maszyny wirtualnej, przechodzimy do parametru **Networking** i zmieniamy wartość domyślną **Not Connected** (brak połączenia) na istniejącą w komputerze hosta kartę sieciową. Sposób przypisania adresu do karty sieciowej systemu wirtualnego zależy ściśle od sposobu adresowania w sieci komputera hosta. Jeśli jest ono statyczne powinniśmy karcie sieciowej systemu gościa również ręcznie przypisać adres. W VMware Server natomiast wybieramy dla maszyny wirtualnej polecenie **Edit virtual machine settings**. Przechodzimy do parametru **Ethernet** i w sekcji **Network Connection** zaznaczamy **Bridged: Connected directly to the physical network**. Co ciekawe w czasie instalacji programu VMware Server do systemu dodawane są wirtualne karty sieciowe. Dzięki temu możemy również zaznaczyć opcję **Custom: Specific virtual network** i z listy wybrać wirtualny adapter **VMnet0 (default Bridged)**. Zachowanie interfejsu VMnet0 można określić w programie Manage Virtual Networks. W zakładce **Host Virtual Network Mapping** tego programu wyświetlana jest lista możliwych do obsłużenia adapterów. Domyślnie interfejs VMnet0 przypisaną ma wartość, dzięki której będzie on wskazywał na aktywną kartę sieciową komputera fizycznego. Możemy jednak wybrać z listy dowolną istniejącą w komputerze hoście kartę sieciową wiążąc ją na stałe z interfejsem VMnet0. Dzięki temu otrzymamy taki sam efekt jak w ustawieniach Virtual PC. Warto też zajrzeć na zakładkę **Automatic Bridging** gdzie możemy między ustalić wyjątki, czyli wybrać adaptery, które nie będą mogły być używane dla opcji bezpośredniego podłączania maszyny wirtualnej do karty sieciowej hosta.

Drugi sposób konfiguracji połączeń sieciowych polega na zbudowaniu własnej podsieci. Maszyna wirtualna będzie wówczas izolowana od istniejącej już sieci (zwiększone bezpieczeństwo) i nie będzie miała dostępu do Internetu (można to oczywiście zmienić poprzez włączenie współdzielenia połączenia internetowego po stronie komputera hosta). Zadanie polega na tym, aby system wirtualny i rzeczywisty miały adresy kart z tego samego zakresu, przy braku modyfikacji ustawień dla istniejących kart sieciowych. Rozwiązanie jest bardziej skomplikowane niż bezpośrednie podłączenie do aktywnej karty sieciowej. Jeśli korzystamy z Virtual PC musimy zainstalować w systemie komputera hosta [kartę połączenia zwrotnego]({{< ref "loopa-tv" >}}). Karta ta umożliwia budowanie sieci wirtualnych. W Windows XP w **Panelu sterowania** wybieramy **Dodaj sprzęt** i klikamy **Dalej**. Uruchomiony kreator będzie starał się znaleźć dodatkowy sprzęt w naszym komputerze. Następnie wyświetlony zostanie formularz, w którym zaznaczamy, iż szukany sprzęt jest już zainstalowany i przechodzimy do następnego okna. Z listy wybieramy **Dodaj nowe urządzenie sprzętowe** i przechodzimy dalej. Następnie zaznaczamy opcję, która umożliwi wybranie sprzętu ręcznie z listy. W kolejnych oknach wybieramy jako sprzęt karty sieciowe, producenta Microsoft i Kartę Microsoft Loopback. W przypadku VMware Server wszystkie niezbędne karty są już przez tę aplikację zainstalowane.

Mając już zainstalowane odpowiednie adaptery sieciowe, musimy je teraz skonfigurować. Zacznijmy od Virtual PC. W ustawieniach maszyny wirtualnej parametr **Networking** powinien przyjąć wartość **Karta Microsoft Loopback**. W **Połączeniach sieciowych** komputera fizycznego, we właściwościach karty Loopback, a następnie protokołu internetowego TCP/IP zaznaczamy opcję **Użyj następującego adresu IP**. Teraz musimy wpisać adres sieciowy. Najlepiej, aby był on z zakresu przewidzianego dla sieci wewnętrznych i jednocześnie różnił się od obecnego adresu komputera fizycznego. Wpiszmy 192.168.13.1 z maską 255.255.255.0. Mając już ustalony adres po stronie komputera fizycznego musimy skonfigurować interfejs sieciowy maszyny wirtualnej. Poleceniem **Start -> Control Panel -> Network Connections -> Local Area Connection -> Properties** otwieramy okno, gdzie zaznaczamy **Internet Protocol (TCP/IP)** i klikamy **Properties**. Tak samo jak na komputerze hoście zaznaczamy opcję **Use the following IP address** i wpisujemy adres 192.168.13.2 z maską 255.255.255.0. Jak widzimy, jedyna różnica jest w ostatniej cyfrze. Jednym słowem mamy dwa komputery, jeden fizyczny a drugi wirtualny, które pracują w jednej sieci 192.168.13.y gdzie y oznacza w naszym przypadku konkretny interfejs. Wywołanie w konsoli komputera hosta polecenia

```dos
ping 192.168.13.2
```

badającego dostępność karty sieciowej maszyny wirtualnej zwróci nam informacje o niemożności połączenia się. Jest to spowodowane poziomem zabezpieczeń systemu gościa. Aby to zmienić należy poleceniem **Start -> Control Panel -> Windows Firewall** uruchomić okno konfiguracyjne ściany ogniowej maszyny wirtualnej. W zakładce **Advanced** w sekcji **ICMP** klikamy przycisk **Settings**, zaznaczamy opcję **Allow incoming echo request** i zatwierdzamy. Po tych zmianach wywołanie polecenia ping powinno zwrócić informacje o czasie wymiany pakietów pomiędzy konfigurowanymi systemami.

{{< figure src="images/ping.png" title="Sprawdzenie dostępności karty sieciowej maszyny wirtualnej" >}}

Jeśli adres istniejącego połączenia sieciowego komputera hosta również jest w formacie 192.168.x.y i x wynosi 13, to aby uniknąć konfliktów z adresacją dwóch sieci karcie Loopback przypiszmy zamiast 13 dowolną inną liczbę np. 14. Aby zorientować się co do przypisanych adresów IP należy uruchomić konsolę systemu komputera rzeczywistego i wpisać polecenie:

```dos
ipconfig /all
```

Zostaną wówczas wyświetlone wszystkie informacje dotyczące konfiguracji istniejących połączeń sieciowych. Poniższy zrzut ekranu pokazuje konfigurację interfejsów dla maszyny wirtualnej. Mniej więcej podobne informacje, tylko w większej ilości, będą wyświetlone po wydaniu polecenie ipconfig dla komputera rzeczywistego.

{{< figure src="images/ipconfig.png" title="Konfiguracja połączeń sieciowych maszyny wirtualnej" >}}

W przypadku WMware Server parametr **Ethernet** maszyny wirtualnej powinien przyjąć wartość **Host-only: A private network shared with host** lub **Custom: Specific virtual network** z wybranym adapterem zawierającym w nazwie host-only np. **VMnet1 (Host-only)**. Ostatni sposób daje też oczywiście więcej możliwości jeśli chodzi o konfigurację działania sieci. Następnie w programie Manage Virtual Networks w zakładce **Summary** sprawdzamy jaki adres przypisany jest do interfejsu odpowiedzialnego za tworzenie sieci prywatnych.

{{< figure src="images/Manage_Virtual_Networks_Summary.png" title="Aplikacja Manage Virtual Networks" >}}

Kolejny krok to konfiguracja karty sieciowej maszyny wirtualnej. Powyższy zrzut ekranu pokazuje, iż przypisany po stronie komputera fizycznego adres to 192.168.157.0 z maską 255.255.255.0. Podobnie jak w przypadku konfiguracji systemu gościa uruchamianego w ramach Virtual PC, tak i teraz przypisujemy do karty sieciowej maszyny wirtualny adres z tej samej sieci. W tym przypadku 192.168.157.2. Po odblokowaniu ściany ogniowej, możemy korzystając z polecenia ping sprawdzić dostępność systemu gościa.

### Internet Information Services

Domyślnie Windows Server 2003 nie posiada zainstalowanych usług internetowych. Dodanie IIS możemy wykonać na dwa sposoby.

W pierwszym, po wybraniu **Start -> Control Panel -> Add or Remove Programs -> Add/Remove Windows Components** zaznaczamy **Microsoft .NET Framework 2.0** a w ramach części **Application Server** element **ASP.NET**, który aktywuje wszystkie niezbędne składniki.

Drugi sposób polega na wybraniu polecenia **Start -> Manage Your Server**. W części **Adding Roles to Your Server** klikamy **Add or remove a role**. Wyświetli nam się okno dialogowe, w których wybieramy **Next** a następnie zaznaczamy opcję **Application Server (IIS, ASP.NET)** i ponownie **Next**. Aktywujemy **ASP.NET** i klikamy dwa razy **Next**. Niestety ta metoda nie instaluje platformy .NET w wersji 2.0 tylko w wersji 1.1. Musimy więc wersję 2.0 tak czy siak jeszcze dodać.

W czasie instalowania składników system poprosi nas o włożenie do napędu CD-ROM płyt instalacyjnych. Cała niezbędna zawartość znajduje się w katalogu **C:\WindowsInstallationFiles\I386** oraz **C:\Documents and Settings\Administrator\WindowsInstallationFiles\CMPNENTS\R2** maszyny wirtualnej.

Zanim jednak przystąpimy do sprawdzania, czy poziomu hosta połączymy się z serwerem WWW maszyny wirtualnej warto poinformować ścianę ogniową serwera, iż chcemy zezwolić na dostęp do IIS z zewnątrz. Poleceniem **Start -> Control Panel -> Windows Firewall** uruchamiamy okno konfigurujące. Przechodzimy na zakładkę **Advanced** i w części **Network Connection Settings** klikamy przycisk **Settings...**. W oknie dialogowym, które się pojawi pozostaje nam zaznaczyć **Web Server (HTTP)** i potwierdzić nasz wybór w kolejnym oknie.