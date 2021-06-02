---
title: Y2K atakuje po latach
date: 2008-09-25T15:04:00+02:00
languageCode: pl-pl
description: Poprawne zapisywanie roku w pliku DBF
categories:
  - Development
tags:
  - .NET
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/09/25/y2k-atakuje-po-latach.aspx
disqus_disable: true
---

Niektórzy pewnie pamiętają mój wpis dotyczący [plików DBF]({{< ref "dbf-po-ludzku" >}}). Nagłówek pliku DBF zawiera datę, gdzie na rok, miesiąc i dzień przeznaczone jest po jednym bajcie.

```csharp
    [StructLayout(LayoutKind.Sequential)]
    internal struct Dbf3Header
    {
        public const byte ReservedSize = 20;

        public byte Dbf;
        public byte Year;
        public byte Mounth;
        public byte Day;
        public int RecordCount;
        public ushort HeaderSize;
        public ushort RecordSize;
        [MarshalAs(UnmanagedType.ByValArray, SizeConst = ReservedSize)]
        public byte[] Reserved;
    }
```

W przypadku miesiąca i dnia jest to zakres wystarczający. Niestety jeśli chodzi o rok już nie. W zeszłym stuleciu (rany, ależ to poważnie brzmi) ze względu na niedostatek zasobów systemowych (pamiętacie stwierdzenie Gatesa, iż 640 KB każdemu wystarczy?) ograniczano do minimum przestrzeń dla dat zazwyczaj pamiętając dla roku tylko dwie ostatnie cyfry.

Zamykając plik DBF wypada uaktualnić datę i czas. Fragment kodu poniżej:

```csharp
                    DateTime now = DateTime.Now;
                    header.Year = (byte)(now.Year % 100);
                    header.Mounth = (byte)now.Month;
                    header.Day = (byte)now.Day;
```

Tak to wygląda teraz. Dzięki dzieleniu modulo % do zmiennej `Year` przypisane są jedności i dziesiątki z roku. Jednak jeszcze kilkanaście dni temu zamiast dzielenia modulo miałem w kodzie zwykłe dzielenie. Skutek? Dla roku 2008 zamiast wartości 08 wpisywana była liczba 20. Jednym słowem chwila nieuwagi i padłem ofiarą roku 2000. Kto by pomyślał.