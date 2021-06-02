---
title: MarshalHelper zarządza niezarządzanym
date: 2008-06-04T14:01:00+02:00
languageCode: pl-pl
description: Kod niezarządzany w .NET C# do operacji odczytu i zapisu plików DBF
categories:
  - Development
tags:
  - .NET
  - Interop
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/06/04/marshalhelper-zarz-dza-niezarz-dzanym.aspx
disqus_disable: true
---

W notce dotyczącej [iteratorów]({{< ref "iteratory" >}}) czy też kiedy opisywałem operacje na plikach [DBF]({{< ref "dbf-po-ludzku" >}}) posługiwałem się klasą pomocniczą `MarshalHelper`, która wykonywała wszystkie niezbędne czynności przy konwersji typów z kodu zarządzanego do niezarządzanego i vice versa. Wykorzystanie klas przestrzeni `System.Runtime.InteropServices` było konieczne, ponieważ w kodzie zarządzanym nie mamy gwarancji, iż pola z danej struktury będą w pamięci ułożone w tej samej kolejności i wyrównane powiedzmy do 8 bajtów.

Sama implementacja metod pomocniczych klasy nie jest skomplikowana. W sieci można znaleźć trochę pomysłów. Poniżej kod całej klasy:

```csharp
    internal static class MarshalHelper
    {
        public static Encoding encoding = Encoding.GetEncoding(852);

        /// <summary>
        /// Zapisanie struktury w pamięci do tablicy bajtów.
        /// </summary>
        /// <param name="structure">Struktura lub klasa, którą chcemy
        /// zapisać.</param>
        /// <returns>Strumień bajtów zawierający strukturę danych.</returns>
        public static byte[] ToByteArray<T>(T structure)
        {
            byte[] buffer = new byte[Marshal.SizeOf(structure)];
            GCHandle handle = GCHandle.Alloc(buffer, GCHandleType.Pinned);
            Marshal.StructureToPtr(structure, handle.AddrOfPinnedObject(), false);
            handle.Free();
            return buffer;
        }

        /// <summary>
        /// Zapianie łańcucha znaków do tablicy bajtów z uwzględnieniem
        /// kodowania (zmiany strony kodowej).
        /// </summary>
        /// <param name="field">Łańcuch, który będziemy zapisywać.</param>
        /// <param name="fieldLength">Liczba bajtów tablicy.</param>
        /// <returns>Tablica bajtów zawierająca łańcuch po operacji
        /// zmiany strony kodowej.</returns>
        public static byte[] StringToByteArray(string field, int fieldLength)
        {
            Debug.Assert(fieldLength > 0);

            if(field == null){
                field = string.Empty;
            }
            byte[] source = encoding.GetBytes(field);
            byte[] destination = new byte[fieldLength];
            Debug.Assert(source.Length <= fieldLength);
            Array.Copy(source, destination, source.Length);
            return destination;
        }

        /// <summary>
        /// Zapisanie tablicy znaków jako łańcucha znaków z uwzględnieniem zmiany
        /// strony kodowej.
        /// </summary>
        /// <param name="field">Tablica znaków, którą będziemy zapisywać.</param>
        /// <returns>Łańcuch zawierający tablicę znaków z uwzględnieniem
        /// zmiany strony kodowej.</returns>
        public static string ByteArrayToString(byte[] field)
        {
            string s = encoding.GetString(field, 0, field.Length);
            int index = s.IndexOf('\0');
            return index > 0 ? s.Remove(index, s.Length - index) : s;
        }

        /// <summary>
        /// Odczytanie struktury ze strumienia.
        /// </summary>
        /// <typeparam name="T">Typ struktury.</typeparam>
        /// <param name="stream">Strumień, z którego czytamy.</param>
        /// <returns>Zwracana struktura.</returns>
        public static T FromStream<T>(Stream stream)
        {
            Debug.Assert(stream != null);

            byte[] buffer = new byte[Marshal.SizeOf(typeof(T))];
            int bytesRead = stream.Read(buffer, 0, buffer.Length);
            if(bytesRead == 0){
                return default(T);
            }
            if(bytesRead != buffer.Length){
                throw new EndOfStreamException(
                    string.Format(
                        "Rozmiar bufora: {0}. Liczba bajtów odczytanych: {1}.",
                        buffer.Length, bytesRead));
            }
            GCHandle handle = GCHandle.Alloc(buffer, GCHandleType.Pinned);
            T structure = (T) Marshal.PtrToStructure(handle.AddrOfPinnedObject(), typeof(T));
            handle.Free();
            return structure;
        }

        /// <summary>
        /// Zapisanie struktury do strumienia.
        /// </summary>
        /// <typeparam name="T">Typ struktury.</typeparam>
        /// <param name="stream">Strumień, do którego zapisujemy.</param>
        /// <param name="structure">Zapisywana struktura</param>
        public static void ToStream<T>(Stream stream, T structure)
        {
            Debug.Assert(stream != null);

            byte[] buffer = ToByteArray(structure);
            stream.Write(buffer, 0, buffer.Length);
        }
    }
```

Życzę miłej zabawy.