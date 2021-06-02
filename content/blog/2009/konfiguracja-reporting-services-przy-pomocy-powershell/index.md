---
title: Konfiguracja Reporting Services przy pomocy PowerShell
date: 2009-02-20T14:42:00+01:00
languageCode: pl-pl
description: Konfiguracja struktury i uprawnień raportów Reporting Services przy pomocy skryptów PowerShell
categories:
  - Databases
tags:
  - PowerShell
  - Reporting Services
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/02/20/konfiguracja-reporting-services-przy-pomocy-powershell.aspx
disqus_disable: true
---

Od pewnego czasu mam przyjemność budować od podstaw system raportowy w jednej z firm finansowych. Dzięki temu możliwe jest przejście przeze mnie całej ścieżki związanej z instalacją, konfiguracją serwera i uprawnień, tworzeniem raportów oraz ich zarządzaniem.

Pierwsze czynności są zawsze takie same. Trzeba pogrupować raporty według określonych przez właściciela biznesowego kategorii oraz nadać uprawnienia dostępu do poszczególnych raportów. Najłatwiej powiązać kategorie z działami występującymi w danej firmie oraz nałożyć uprawnienia dostępu na poziomie folderów. Oczywiście z biegiem czasu, kiedy raportów powstaje coraz więcej i rośnie świadomość użytkowników, zaczynają się pojawiać żądania dotyczące modyfikacji uprawnień, dostępu grup do folderów lub poszczególnych raportów. Przy dużej liczbie raportów i dużej liczbie tego typu żądań zarządzanie uprawnieniami zaczyna stawać się zadaniem, któremu może się okazać, iż poświęcamy zbyt wiele czasu.

## Ograniczenia

Załóżmy, iż na serwerze raportów mamy zdefiniowaną następującą strukturę folderów oraz zawartych w nich raportów:

{{< figure src="images/FolderyRaporty.png" title="Przykładowa struktura folderów i raportów" >}}

Do folderu **Folder_1** dostęp ma grupa użytkowników **Grupa_1**, która jednocześnie nie ma dostępu do folderu **Folder_2**. W jaki sposób **Grupa_1** ma mieć zrealizowany dostęp do raportu **Raport_2_3**?

Pierwszy sposób do zrobienie skrótu (**link**) w **Folder_1** raportu **Raport_2_3**. Ale w zaprezentowanej strukturze **Raport_2_3** korzysta z podraportu **Podraport_2_3_1** oraz wywołuje raport **Raport_2_2**. To oznacza, iż podraport oraz odniesienie do raportu **Raport_2_2** w folderze **Folder_1** nie będą działać z powodu braku uprawnień. Czy zrobienie w związku z tym skrótu do **Podraport_2_3_1** i **Raport_2_2** w folderze **Folder_1** rozwiąże nam problem? Niestety nie. **Raport_2_3** dalej będzie się odwoływać do zawartości folderu **Folder_2**.

Jak inaczej można rozwiązać ten problem? Drugi sposób polega na wgraniu interesujących nas raportów całkowicie od nowa do właściwego folderu. Tylko, że takie podejście powoduje, iż przy dużej ilości takich zależności zarządzanie uaktualnianiem definicji raportu staje się pracochłonne i podatne na błędy – trzeba bowiem uaktualnić wszystkie wersje danego raportu w systemie.

Trzeci sposób wiąże się ze zmianą zarządzania uprawnieniami. Zamiast zarządzać dostępem na poziomie folderów należy przejść na poziom poszczególnych raportów. Problem w tym, iż uprawnienia folderu nadrzędnego to suma uprawnień wszystkich raportów i podfolderów. Jest to konieczne aby użytkownik mógł dostać się do folderu w celu przeglądania dostępnych raportów. Przez to łatwiejszym staje popełnienie błędu i udostępnienie dowolnego raportu wszystkim grupom, które mają dostęp do danego folderu.

Ze skrótami do raportów wiąże się jeszcze jeden ważny problem. Załóżmy, iż z jakiś względów do raportów w folderze **Folder_2** dostęp uzyskuje grupa **Grupa_1**. Uprawnienia są przyznawane na poziomie folderu. Następnie w folderze **Folder_2** tworzony jest skrót do raportu **Raport_3_1** z folderu **Folder_3**. Efektem tych zmian jest możliwość wywoływania raportu **Raport_3_1** przez grupę **Grupa_1** mimo, iż może wcale nie to było naszym celem.

## Wybór rozwiązania

Co w takim razie powinniśmy zrobić aby ogarnąć temat konfiguracji uprawnień? To co mnie się od razu nasunęło było skorzystanie ze skryptu. Nad wyborem języka skryptowego niewiele się zastanawiałem. Wybór PowerShell’a był w sumie dość oczywisty. Pozostało jedynie wybrać sposób komunikacji z  usługami raportującymi. Pierwsza możliwość to skorzystanie z programu **rs.exe** dostępnego po instalacji Reporting Services. Aplikacja ta nie umożliwia jednak wykonywania bardziej zaawansowanych czynności przez co nie będziemy mogli z niej skorzystać. A sposób drugi?

Usługi raportujące składają się z dwóch aplikacji webowych:

* **ReportManager**;
* **ReportServer**.

Użytkownik końcowy najczęściej korzysta z **ReportManager** udostępnianej pod nazwą **Reports**. Aplikacja ta umożliwia między innymi przeglądanie i wyświetlanie raportów w ramach przeglądarki internetowej. **ReportServer** dostarcza natomiast usługi sieciowe wykorzystywane przez **ReportManager** w celu pobierania, wyświetlania i modyfikowania zawartości bazy danych serwera raportów.

Usługi sieciowe i PowerShell. Czemu nie…

## PowerShell i Policy

Kilka słów na temat zabezpieczeń PowerShell. Domyślnie po instalacji można uruchamiać jedynie skrypty podpisane. Poziom uprawnień można sprawdzić wpisując w konsoli PowerShell polecenie `Get-ExecutionPolicy`. Dozwolone wartości to:

* `Restricted`;
* `AllSigned`;
* `RemoteSigned`;
* `Unrestricted`.

Domyślne ograniczenie może być dla nas zbyt bolesne. Dlatego też jeśli mamy ustawiony poziom zabezpieczeń jako `Restricted` lub `AllSigned` możemy go zmienić na mniej restrykcyjny:

```powershell
Set-ExecutionPolicy RemoteSigned
```

Istnieje również możliwość skorzystania z **Group Policy** o czym można przeczytać na stronach [WindowSecurity.com](http://www.windowsecurity.com/articles/PowerShell-Security.html).

## Parametry startowe

Rozpoczęcie wykonywania skryptu rozpoczynamy między innymi od zdefiniowania stałych:

```powershell
    # Adres serwera raportów
    [string] $reportServerAddress = "http://localhost/reportserver/reportservice2005.asmx?WSDL"

    # Miejsce składowania definicji raportów
    [string] $reportProjectFolder = "C:\Raporty\src"

    # Zmienne zawierające źródła danych wymaganych przez raporty
    [string] $dataSourceReferenceName = "/Data Sources/ReportsDataSource"

    # Miejsce nadrzędne dla konfigurowanych raportów. Katalog startowy
    [string] $global:root = "/"
```

oraz zmiennych globalnych:

```powershell
    $global:assembly = $null
    $global:proxy = $null
```

Adres serwera raportów zawsze będzie taki sam. Jedynie w przypadku konfiguracji zdalnej serwera raportów nazwę **localhost** należy zastąpić nazwą lub adresem właściwego komputera. Jeśli chodzi o źródła danych to zakładam, iż są one już utworzone w ramach usług raportujących. Dzięki temu unikam zapisywania w pliku konfigurującym ścieżek dostępu i haseł do serwerów baz danych. W powyższym przykładzie zdefiniowane jest tylko jedno źródło danych, ale w może być ich (tak jak u mnie w systemie produkcyjnym) oczywiście więcej. Zmienne `$global:assembly` oraz `$global:proxy` będą zawierać klasy umożliwiające zarządzanie serwerem raportów.

## Przygotowanie połączenia z serwerem raportów

Część rozwiązań, między innymi funkcja `New-WebServiceProxy` dostępna od PowerShell V2 (CTP3), zwraca obiekt proxy umożliwiający jedynie wykonywanie operacji udostępnianych przez usługi sieciowe serwera raportów. Jest to za mało, ponieważ potrzebować będziemy również możliwości tworzenia nowych obiektów. Dlatego też jako rozwiązanie właściwe wybrałem propozycję Christiana Glessnera. Szczegóły można znaleźć we wpisie [PowerShell, WebServices & SharePoint](http://cglessner.blogspot.com/2008/07/powershell-webservices-sharepoint.html) na jego blogu. Źródła można pobrać z witryny <http://www.codeplex.com/iLoveSharePoint/Release/ProjectReleases.aspx>.

W swoim rozwiązaniu wykorzystuję zmodyfikowaną wersję metody `Get-WebServiceProxy` ze skryptu Christiana Glessnera:

```powershell
    function Create-WebServiceProxy(
        [string] $url = $(throw 'Brak adresu serwera raportów!'))
    {
        Write-Host "Nawiązywanie połączenia z serwerem $url"

        $fileName = [System.IO.Path]::Combine(
            [System.Environment]::CurrentDirectory,
            "Proxy.ReportingServices2005")
        if(!(Test-Path "$fileName.dll")){
            $WinSDK = "$env:ProgramFiles\Microsoft SDKs\Windows\v6.0A\Bin"
            $Net35 = "$env:SystemRoot\Microsoft.NET\Framework\v3.5"

            $null =& $WinSdk\wsdl.exe $url /n:Proxy /out:"$fileName.cs"
            $null =& $Net35\csc.exe /t:library /out:"$fileName.dll" "$fileName.cs"
        }
        $global:assembly = [System.Reflection.Assembly]::LoadFrom("$fileName.dll")
        $proxyType = $global:assembly.GetTypes() | Where-Object { $_.IsSubclassOf(
            [System.Web.Services.Protocols.SoapHttpClientProtocol]) -eq $true}
        $global:proxy = New-Object -TypeName $proxyType

        Write-Host "Połączenie nawiązane"
    }
```

Jak widzimy, w powyższym kodzie tworzona, ładowana do pamięci i przechowywana w zmiennej globalnej `$global:assembly` jest biblioteka **Proxy.ReportingServices2005.dll** zawierająca wszystkie potrzebne typy. Zmienna globalna `$global:proxy` zawiera instancję klasy umożliwiającej odpytywanie usług sieciowych serwera raportowego znajdującego się w określonej przez nas lokalizacji.

Utworzenie biblioteki **Proxy.ReportingServices2005.dll** wymaga **Microsoft SDKs**, które wgrywane do systemu jest automatycznie w czasie instalacji Visual Studio. Jeśli z jakiś przyczyn na serwerze, na którym uruchamiamy powyższy skrypt nie ma i nie może być zainstalowane **Microsoft SDKs**, należy bibliotekę **Proxy.ReportingServices2005.dll** utworzyć na komputerze programisty, a następnie przekopiować razem ze skryptem w miejsce docelowe. Rozwiązanie zadziała. Dzięki warunkowi `if(!(Test-Path "$fileName.dll"))` biblioteka nie będzie ponownie tworzona jeśli już istnieje.

Czas na przygotowanie połączenia z serwerem raportów:

```powershell
    Create-WebServiceProxy $reportServerAddress
    $global:proxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials
```

Koniecznie musimy pamiętać o przypisaniu danych uprawnionego użytkownika. W przypadku kiedy konfigurujemy **localhost** wystarczy przypisać aktualnie zalogowanego do komputera użytkownika – z definicji użytkownicy lokalni mają bowiem uprawnienia administracyjne w ramach usług raportowych.

## Konfiguracja folderu startowego

Po nawiązaniu połączenia z serwerem raportów czyścimy uprawnienia głównego katalogu, w ramach którego będziemy tworzyć foldery i wgrywać raporty.

```powershell
    $policies = Create-Policy $adminGroups "WAW-RS\ReportUser"
    $global:proxy.SetPolicies("$root", $policies)
```

Funkcja `Create-Policy` na podstawie przekazanych tablic zawierających łańcuchy z nazwami uprawnień tworzy odpowiednią kolekcję obiektów, którą możemy przekazać do usługi sieciowej **SetPolicies**. Zmienna `$adminGroups` zawiera minimalne uprawnienia administratora:

```powershell
    $adminGroups = "BUILTIN\Administrators"
```

jeśli system, na którym uruchamiamy skrypt jest w angielskiej wersji językowej lub:

```powershell
    $adminGroups = "BUILTIN\Administratorzy"
```

jeśli system jest w polskiej wersji językowej. W powyższym przykładzie do metody `Create-Policy` przekazujemy również uprawnienia do przeglądania raportów dla użytkownika **"WAW-RS\ReportUser"**. Serwer raportów może bowiem zawierać raporty dla innych aplikacji webowych, których nie będziemy konfigurować. Ważne jest, aby uprawnienia przekazywane do funkcji `Create-Policy` w skrypcie podawać zawsze z nazwą komputera!

Co natomiast robi funkcja `Create-Policy`? Oto jej kod:

```powershell
    function Create-Policy([string[]] $adminGroups, [string[]] $browseGroups)
    {
        [Proxy.Policy[]] $policies = Create-AdminPolicy $adminGroups
        foreach($browseGroup in $browseGroups){
            $role = New-Object "Proxy.Role"
            $role.Name = "Browser"
            $policy = New-Object "Proxy.Policy"
            $policy.GroupUserName = $browseGroup
            $policy.Roles += $role
            $policies += $policy
        }
        return $policies
    }
```

Oraz kod metody `Create-AdminPolicy`:

```powershell
    function Create-AdminPolicy([string[]] $adminGroups)
    {
        [Proxy.Policy[]] $policies = @()
        foreach($adminGroup in $adminGroups){
            $role = New-Object "Proxy.Role"
            $role.Name = "Content Manager"
            $policy = New-Object "Proxy.Policy"
            $policy.GroupUserName = $adminGroup
            $policy.Roles += $role
            $policies += $policy
        }
        return $policies
    }
```

W funkcjach `Create-Policy` oraz `Create-AdminPolicy` dokonuję pewnych założeń i uproszczeń. Otóż każdej grupie przekazanej przez zmienną `$browseGroup` przypisywane jest uprawnienie **"Browser"**. Usługi raportujące mają więcej w tej materii możliwości, ale ponieważ nie korzystam z nich nie są przeze mnie w  skrypcie obsługiwane.

## Konfiguracja folderów i raportów

Proces konfiguracji będzie przebiegał według następującego schematu:

* Utworzenie jeśli konieczne folderu dla raportów;
* Nadanie niezbędnych uprawnień do folderu;
* Wgranie jeśli konieczne pliku rdl z definicją raportu;
* Konfiguracja właściwości raportu;
* Przypisanie źródła danych do raportu;
* Konfiguracja uprawnień dostępu do raportu;
* Dodanie do folderów nadrzędnych uprawnień koniecznych do dostępu do raportu.

Przykład załadowania i konfiguracji kilku kombinacji raportów:

```powershell
    Write-Host
    Write-Host "Konfigurowanie raportów przykładowych"
    $folder = "Raporty przykładowe"
    $reportPath = "$reportProjectFolder\Raporty przykładowe"
    $browseGroups = "AD\Grupa1", "AD\Grupa2", "AD\Grupa3"

    Create-Folder $folder $root $adminGroups
    $path = Combine-Path $root $folder

    Create-Report "$path" "$reportPath\Raport 1.rdl" $adminGroups
        $browseGroups $dataSourceReferenceName
    Create-Report "$path" "$reportPath\Raport 1 - podraport.rdl" $adminGroups
        $browseGroups $dataSourceReferenceName $hide
    Create-Report "$path" "$reportPath\Raport 2.rdl" $adminGroups
        ($browseGroups + "AD\Grupa4") $dataSourceReferenceName
    Create-Report "$path" "$reportPath\Raport 3.rdl" $adminGroups
        ($browseGroups + "AD\Grupa4", "AD\Grupa5") $dataSourceReferenceName
    Create-Report "$path" "$reportPath\Raport 4.rdl" $adminGroups
        $browseGroups $null
```

Stała pomocnicza `$hide` zwraca wartość `$true` i podawana jako parametr funkcji `Create-Report` umożliwia ukrycie raportu.

Po zdefiniowaniu wartości zmiennych zaczynamy od utworzenia i aktualizacji uprawnień folderu. Do tego wykorzystujemy metodę `Create-Folder`:

```powershell
    function Create-Folder([string] $folder, [string] $parent = "/", [string[]] $adminGroups)
    {
        [string] $path = Combine-Path $parent $folder
        if(!(ItemExists $parent $folder)){
            Write-Host "Utworzenie folderu $path"
            $properties = @()
            $global:proxy.CreateFolder($folder, $parent, $properties)
        }
        Write-Host "Usunięcie uprawnień dla folderu $path"
        $policies = Create-AdminPolicy $adminGroups
        $global:proxy.SetPolicies("$path", $policies)
    }
```

Dwie metody pomocnicze wywoływane przez funkcję `Create-Folder` to:

```powershell
    function Combine-Path([string] $path1, [string] $path2)
    {
        [string] $path = [System.IO.Path]::Combine($path1, $path2)
        return $path.Replace("\", "/")
    }
```

umożliwiająca budowanie poprawnych ścieżek dostępu, oraz `ItemExists`, której zadaniem jest sprawdzenie czy w danej lokalizacji istnieje szukany przez nas element: folder, raport, źródło danych...

```powershell
    function ItemExists([string] $folder, [string] $name)
    {
        [Proxy.BooleanOperatorEnum] $operator = 0
        [Proxy.ConditionEnum] $condition = 1
        $searchCondition = New-Object "Proxy.SearchCondition"
        $searchCondition.Condition = $condition
        $searchCondition.ConditionSpecified = $true
        $searchCondition.Name = "Name"
        $searchCondition.Value = $name
        [Proxy.CatalogItem[]] $items =
            $global:proxy.FindItems($folder, $operator, $searchCondition)
        return $items -and ($items.Length -cgt 0)
    }
```

Największe jednak czynności są wykonywane w czasie wgrywania i konfigurowania konkretnego raportu:

```powershell
    function Create-Report([string] $parent, [string] $path,
        [string[]] $adminGroups, [string[]] $browseGroups,
        [string] $dataSourceReferenceName,
        $hide = $false, $overwrite = $false)
    {
        Write-Host
        $name = [System.IO.Path]::GetFileNameWithoutExtension($path)
        $report = "$parent/$name"

        if(!$overwrite){
            if(!(ItemExists $parent $name)){
                Upload-Report $parent $path $name $overwrite
            }
        }

        Set-ReportProperty $report $hide
        Set-ReportReferenceDataSource $report $dataSourceReferenceName
        [Proxy.Policy[]] $policies = Create-Policy $adminGroups $browseGroups
        Set-ReportPolicy $report $name $policies
        Set-FolderPolicy $parent $policies
    }
```

Domyślnie jeśli dany raport już istnieje to nie jest aktualizowana jego definicja. Poprzez modyfikację wartości domyślnej dla parametru `$overwrite` możemy zmienić zachowanie skryptu. Pamiętajmy, iż nie należy usuwać raportów w celu modyfikacji definicji aby nie stracić istniejących historii i subskrypcji. Domyślne zachowanie skryptu jest takie, aby można było go uruchamiać w dowolnym (no prawie) momencie w celu sprawdzenia i korekcji uprawnień.

Funkcja wgrywająca definicję raportu wygląda zaś następująco:

```powershell
    function Upload-Report([string] $parent, [string] $path,
        [string] $name, [bool] $overwrite)
    {
        Write-Host "Utworzenie raportu $report"
        $stream = [System.IO.File]::OpenRead($path)
        $definition = New-Object byte[] $stream.Length
        $stream.Read($definition, 0, $stream.Length)
        $stream.Close()
        $global:proxy.CreateReport($name, $parent,
            $overwrite, $definition, $null)
    }
```

Jak widzimy ułatwiamy sobie życie korzystając z bibliotek platformy .NET!

Właściwości, które ustawiamy są skromne.W chwili obecnej jest to jedynie ukrywanie raportu. Ma to sens na przykład w przypadku podraportów, które nie stanowią jednocześnie samodzielnego raportu:

```powershell
    function Set-ReportProperty([string] $report, [bool] $hidden)
    {
        Write-Host "Aktualizacja właściwości raportu $name"
        $property = New-Object "Proxy.Property"
        $property.Name = "Hidden"
        $property.Value = $hidden
        $properties = @($property)
        $global:proxy.SetProperties($report, $properties)
    }
```

Po ustawieniu właściwości następuje przypisanie źródła danych do raportu. Oczywiście nie każdy raport musi posiadać źródło danych. Stąd warunek na początku funkcji:

```powershell
    function Set-ReportReferenceDataSource([string] $report,
        [string] $dataSourceReferenceName)
    {
        if($dataSourceReferenceName -eq $null -or
            $dataSourceReferenceName.Length -lt 1){
            return
        }

        Write-Host "Aktualizacja źródła danych raportu $name"
        $dataSourceReference = New-Object "Proxy.DataSourceReference"
        $dataSourceReference.Reference = $dataSourceReferenceName
        $dataSources = $global:proxy.GetItemDataSources($report)
        # Poniższe można odkomentować jeśli nie zależy nam na poprawności
        # przypisań źródła danych
        #if($dataSources -eq $null){
        #    return
        #}
        foreach($dataSource in $dataSources){
            $dataSource.Item = $dataSourceReference
        }
        $global:proxy.SetItemDataSources($report, $dataSources)
    }
```

W powyższym kodzie dokonuję kilku ważnych założeń. Po pierwsze jak już wcześniej mówiłem, zakładam, iż źródła danych w ramach usług raportowych są już utworzone. Dlatego też stosuję typ `Proxy.DataSourceReference`. Po drugie każdy raport korzysta docelowo tylko z jednego źródła danych. Nawet jeśli w czasie tworzenia raportu i jego testów wykorzystywałem dwa czy trzy źródła danych to na serwerze produkcyjnym mam tylko jedno źródło. Jeśli to założenie w Waszym przypadku nie będzie prawdziwe należy dokonać modyfikacji funkcji `Set-ReportReferenceDataSource` tak, aby parametr `$dataSourceReferenceName` był tablicą.

Przypisanie uprawnień do raportu:

```powershell
    function Set-ReportPolicy([string] $report, [string] $name,
        [Proxy.Policy[]] $policies)
    {
        Write-Host "Nadanie uprawnień do raportu $name"
        $global:proxy.SetPolicies($report, $policies)
    }
```

I na koniec rekurencyjne ustawienia właściwości folderu zawierającego konfigurowany raport. Znak `%` jest skrótowym zapisem polecenia `Foreach-Object`:

```powershell
    function Set-FolderPolicy([string] $parent, [Proxy.Policy[]] $policies)
    {
        $folderPolicies = @()
        $folderPoliciesInherited = [ref] $false
        $folderPolicies = $global:proxy.GetPolicies($parent,
            $folderPoliciesInherited)
        foreach($policy in $policies){
            if($($folderPolicies | % {$_.GroupUserName}) -notcontains $policy.GroupUserName){
                $folderPolicies += $policy
            }
        }
        Write-Host "Aktualizacja uprawnień folderu $parent"
        $global:proxy.SetPolicies($parent, $folderPolicies)

        if($parent -eq $global:root){
            return
        }

        for($i = $parent.Length; 0, $i--){
            if($parent[$i] -eq "/"){
                $newParent = $parent.Substring(0, $i)
                if($newParent.Length -eq 0){
                    $newParent = "/"
                }
                Set-FolderPolicy $newParent $policies
                break
            }
        }
    }
```

Pamiętajmy o bardzo ważnym fakcie – uprawnienia folderu zawierający podfoldery i raporty są sumą uprawnień podfolderów i raportów!

## Zakończenie

Uzbrojeni w taki skrypt możemy spokojnie wypłynąć na szerokie wody konfiguracyjne usług raportowych. Jedyne problemy jakie mogą nas spotkać dotyczyć będą sytuacji, kiedy będziemy próbowali skorzystać z protokołu https przy braku zaufanego certyfikatu. Nie ma bowiem możliwości potwierdzenia świadomego łączenia się z serwerem.

Osobom, które chciałyby spróbować swoich sił w wykorzystaniu języka PowerShell polecam pobranie darmowego [PowerGUI](http://www.powergui.org/) umożliwiającego pisanie skryptów oraz zarządzanie (!) komputerem i nie tylko przy pomocy hierarchicznie zorganizowanych skryptów PowerShell. Warto również z sekcji [PowerPacks](http://www.powergui.org/kbcategory.jspa?categoryID=21) pobrać pakiet [SQL Server 2005 Reporting Services Power Pack](http://www.powergui.org/entry.jspa?externalID=2045&categoryID=54) umożliwiający zarządzanie usługami raportującymi.

Grafika (schemat folderów) została wykonana przy pomocy <http://bubbl.us>.