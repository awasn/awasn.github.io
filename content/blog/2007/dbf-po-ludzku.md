---
title: DBF po ludzku
date: 2007-06-12T15:02:00+02:00
languageCode: pl-pl
description: Obsługa operacji odczytu i zapisu plików DBF w .NET C#
categories:
  - Development
  - Databases
tags:
  - .NET
  - Interop
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2007/06/12/dbf-po-ludzku.aspx
disqus_disable: true
---

Jakiś czas temu Sławek [pisał](http://zine.net.pl/blogs/ucel/archive/2007/05/14/Dost_1901_p-do-danych-Excela-za-pomoc_0501_-ODBC.aspx) o dostępie poprzez sterowniki ODBC do tabel w formacie Excel. W podobny sposób można również próbować przetwarzać pliki DBF. Ale można też prościej... bardziej po ludzku...

Wersji formatu DBF jest oczywiście wiele, ale my zajmiemy się wersją 3, bardzo popularną zwłaszcza w środowisku MS-DOS. Niezbędne będzie przy tym skorzystanie z przestrzeni nazw `InteropServices` umożliwiającej dostęp do kodu niezarządzanego.

Każdy plik DBF składa się z nagłówka

```csharp
    [StructLayout(LayoutKind.Sequential)]
    struct Dbf3Header
    {
        public const byte ReservedSize = 20;

        public byte Dbf;
        public byte Year;
        public byte Mounth;
        public byte Day;
        public int RecordCount;
        public ushort HeaderSize;
        public ushort RecordSize;
        [MarshalAs(UnmanagedType.ByValArray, SizeConst=Dbf3Header.ReservedSize)]
        public byte[] Reserved;
    }
```

który zawiera podstawowe informacje o przechowywanych wewnątrz danych. Nazwy pól są na tyle dosłowne, iż nie będziemy się na ich temat rozwodzić. Każde pole, które będzie występować w pliku, opisane jest w następujący sposób:

```csharp
    [StructLayout(LayoutKind.Sequential)]
    struct Dbf3Field
    {
        public const byte FieldNameSize = 11;
        public const byte ReservedSize = 14;

        [MarshalAs(UnmanagedType.ByValArray, SizeConst=Dbf3Field.FieldNameSize)]
        public byte[] FieldName;
        public byte FieldType;
        public int Offset;
        public byte Length;
        public byte Precision;
        [MarshalAs(UnmanagedType.ByValArray, SizeConst=Dbf3Field.ReservedSize)]
        public byte[] Reserved;
    }
```

Pole `FieldType` zawiera rodzaj pola i może przyjmować wartość (w wersji tej dla uproszczenia pominiemy pola typu memo):

* `C` dla pola znakowego;
* `D` dla pola data;
* `L` dla pola logicznego;
* `N` dla pola numerycznego.

Aby móc wykonywać operacje na pliku DBF musimy go wpierw otworzyć:

```csharp
        private void OpenDbf()
        {
            header = MarshalHelper.FromStream<Dbf3Header>(this.stream);
            int fieldCount =
                (header.HeaderSize - Marshal.SizeOf(header)) /
                Marshal.SizeOf(typeof(Dbf3Field));

            fields = new Dictionary<string, Dbf3Field>(fieldCount);
            int offset = 1;
            for (int i = 0; i < fieldCount; i++)
            {
                Dbf3Field field = MarshalHelper.FromStream<Dbf3Field>(
                    this.stream);
                field.Offset = offset;
                offset = offset + field.Length;
                fields.Add(MarshalHelper.ByteArrayToString(
                    field.FieldName).ToUpper(), field);
            }

            buffer = new byte[header.RecordSize];
        }
```

lub utworzyć:

```csharp
        private void CreateDbf()
        {
            header.Dbf = 0x03;
            header.RecordCount = 0;
            header.RecordSize = 0;
            header.HeaderSize = (ushort)(Marshal.SizeOf(header) + 1);

            MarshalHelper.ToStream<Dbf3Header>(stream, header);
            this.WriteToStreamEndOfHeader();

            fields = new Dictionary<string, Dbf3Field>();
        }
```

Jeśli plik został przez nas utworzony to powinniśmy dodać do niego definicje pól, które będą wewnątrz składowane (w poniższym przykładzie dla uproszczenia pominięto wszystkie pola poza rodzajem data):

```csharp
        public void NewField(string fieldName, Dbf3FieldType fieldType,
            byte length, byte precision)
        {
            Dbf3Field field = new Dbf3Field();

            field.FieldName = MarshalHelper.StringToByteArray(
                fieldName.ToUpper(), Dbf3Field.FieldNameSize);
            switch (fieldType)
            {
                ...
                case Dbf3FieldType.Date:
                    field.FieldType = Convert.ToByte('D');
                    field.Length = 8;
                    field.Precision = 0;
                    break;
                ...
            }

            if (header.RecordSize == 0)
                header.RecordSize = 1;
            field.Offset = header.RecordSize;
            header.HeaderSize += (ushort)Marshal.SizeOf(typeof(Dbf3Field));
            header.RecordSize += field.Length;

            fields.Add(fieldName.ToUpper(), field);
        }
```

Mając przygotowany plik DBF możemy dodawać do niego nowe wiersze. Wpierw przygotowujemy bufor dla danych:

```csharp
        public void NewRecord()
        {
            buffer = new byte[header.RecordSize];
        }
```

a po wypełnieniu zapisujemy na dysk:

```csharp
        public void Write()
        {
            stream.Position = header.HeaderSize +
                header.RecordSize * header.RecordCount;
            header.RecordCount++;
            stream.Write(buffer, 0, buffer.Length);
        }
```

Wszystkie dane w plikach DBF są składowane w postaci łańcuchów. Oznacza to, iż tak naprawdę potrzebujemy jednej głównej metody do zapisu danych:

```csharp
        public void SetString(string fieldName, string value)
        {
            Dbf3Field field = this.GetField(fieldName);
            string fieldValue;
            if (value == null)
                value = string.Empty;
            if (field.FieldType == Convert.ToByte('C'))
                fieldValue = value.PadRight(field.Length);
            else
                fieldValue = value.PadLeft(field.Length);
            byte[] data = Encoding.Default.GetBytes(fieldValue);
            Buffer.BlockCopy(data, 0, buffer, field.Offset, field.Length);
        }
```

Reszta metod umożliwiających zapisywanie innych typów danych wywołuje wewnętrznie tę metodę wykonując rzutowanie na typ łańcuchowy np.:

```csharp
        public void SetInt32(string fieldName, int value)
        {
            this.SetString(fieldName, Convert.ToString(
                value, dbfCultureInfo));
        }
```

Przy konwersji musimy zwrócić uwagę na fakt, iż dane w pliku DBF są składowane według amerykańskich ustawień regionalnych. To samo dotyczy danych odczytywanych:

```csharp
        public int GetInt32(string fieldName)
        {
            return Convert.ToInt32(this.GetString(fieldName),
                dbfCultureInfo);
        }
```

Operując na plikach DBF należy również pamiętać, iż po nagłówku należy w strumieniu umieścić znacznik końca nagłówka `0x0D`, a po zapisaniu wszystkich danych na dysk znacznik końca danych `0x1A`.

Powyższe przykłady zawierają między innymi wywołania metod pomocniczej klasy `MarshalHelper`. Klasa ta definiuje metody ułatwiające zamianę kodu zarządzanego na niezarządzany i odwrotnie. Korzysta ona przede wszystkim z `Marshal.StructureToPtr` oraz z `Marshal.PtrToStructure`.

Dzięki powyższemu kodowi możemy uniezależnić się od sterowników i ustawień systemowych co niejednokrotnie bardzo się przydaje, jeśli oczywiście ktoś z tego typu rozwiązań musi korzystać.

---

* Klasa `MarshalHelper` dostępna jest (wraz z opisem) we wpisie z czerwca 2008 roku: [MarshalHelper zarządza niezarządzanym]({{< ref "marshalhelper-zarzadza-niezarzadzanym.md" >}})