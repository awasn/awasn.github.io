---
title: Kiedy nie działa tryb zgodności w Windows 7
date: 2009-05-20T10:48:00+02:00
lastmod: 2018-09-17T13:59:00+02:00
languageCode: pl-pl
description: Tryb zgodności Windows w przypadku jeśli aplikacja samodzielnie sprawdza wersję systemu i od tego uzależnia swoją dalszą pracę
categories:
  - Development
tags:
  - .NET
  - Interop
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/05/20/kiedy-nie-dzia-a-tryb-zgodno-ci-w-windows-7.aspx
disqus_disable: true
---

W razie problemów z działaniem aplikacji w systemie Windows 7 można we właściwościach danego programu (**Properties –> Comaptibility**) włączyć tryb zgodności (**Compatibility mode**) poprzez wybranie wcześniejszej wersji systemu operacyjnego. Do dyspozycji mamy:

* Windows 95;
* Windows 98 / Windows Me;
* Windows NT 4.0 z Service Pack 5;
* Windows 2000;
* Windows XP z Service Pack w wersji 2 lub 3;
* Windows Server 2003 z Service Pack 1;
* Windows Vista;
* Windows Vista z Service Pack 1 lub 2.

Rozwiązanie to jednak nie zadziała poprawnie jeśli aplikacja samodzielnie sprawdza wersję systemu i od tego uzależnia swoją dalszą pracę. Weźmy dla przykładu poniższy kod:

```csharp
    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
    public class OSVERSIONINFO
    {
        public int OSVersionInfoSize;
        public int MajorVersion;
        public int MinorVersion;
        public int BuildNumber;
        public int PlatformId;
        [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 128)]
        public string CSDVersion;
    }

    class Program
    {
        [DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
        public static extern bool GetVersionEx([In, Out] OSVERSIONINFO ver);

        static void Main()
        {
            var structure = new OSVERSIONINFO();
            structure.OSVersionInfoSize = Marshal.SizeOf(structure);
            GetVersionEx(structure);
            Console.WriteLine(string.Format("Wersja systemu: {0}.{1}",
                structure.MajorVersion,
                structure.MinorVersion));
            Console.ReadKey();
        }
    }
```

Jaki system nie wybierzemy w trybie zgodności, dla Windows 7 RC wynik zawsze będzie identyczny: **Wersja systemu: 6.1** (podobnie zresztą jak dla Windows Server 2008 R2).

PS. Z powyższych przyczyn w Windows 7 póki co nie działa między innymi WD Anywhere Backup. Jest to więc kwestia aplikacji a nie problemów z samym systemem operacyjnym.