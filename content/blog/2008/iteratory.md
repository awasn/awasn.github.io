---
title: Iteratory
date: 2008-05-26T23:06:00+02:00
lastmod: 2018-09-17T12:18:00+02:00
languageCode: pl-pl
description: Mniejsze zużycie pamięci .NET przy korzystaniu z iteratorów
categories:
  - Development
tags:
  - .NET
  - .NET Compact Framework
  - Interop
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/05/26/iteratory.aspx
disqus_disable: true
---

Mój pierwszy blog znajdował się na portalu `developers.pl`. Niestety z różnych przyczyn serwis ten padł. A szkoda. Choćby dlatego, iż miałem tam kilka ciekawych wpisów. Nie chciałbym aby zostały one wszystkie stracone dlatego też postanowiłem jeden z nich przypomnieć (również sobie). Ciekawa była optymalizacja pierwotnego kodu w ramach komentarzy do tej notki...

Jedną z nowości .NET w wersji 2.0 są (dzisiaj już możemy powiedzieć, że były) iteratory. Dzięki nim przekazywanie kolekcji obiektów czy struktur pomiędzy klasami nie musi już oznaczać zajmowania nowych, pomocniczych obszarów w pamięci, co konieczne było zwłaszcza jeśli nasze oprogramowanie zbudowane było w oparciu o warstwy (dostępu do danych, logiki biznesowej czy prezentacji).

Załóżmy, iż tworzymy rozwiązanie, które będzie zapisywało różne obiekty do pliku. Strumień z danymi wyjściowymi będzie zawsze ten sam, ale dane wejściowe możemy odbierać różnymi kanałami: z bazy danych, innych plików itd. Jedna z zapisywanych klas wyglądać będzie następująco:

```csharp
    [StructLayout(LayoutKind.Sequential, Pack = 4)]
    public class ProductCategory
    {
        public const byte CodeSize = 21;
        public const byte NameSize = 41;

        public int Id;

        [MarshalAs(UnmanagedType.ByValArray, SizeConst = ProductCategory.CodeSize)]
        public byte[] Code;

        [MarshalAs(UnmanagedType.ByValArray, SizeConst = ProductCategory.NameSize)]
        public byte[] Name;
    }
```

Zamiast klas można również próbować wykorzystać struktury. Okazuje się jednak, iż nie jest to dobry sposób, zwłaszcza jeśli będziemy korzystać z pętli `foreach`. Powodem są operacje **boxing** i **unboxing** (czyli rzutowania w obie strony pomiędzy klasą `Object` a naszym typem), które wykonywane są w czasie przechodzenia po kolejnych elementach dla **value type**, do których poza typami prymitywnymi należą właśnie struktury. Może się okazać, iż straty na tych operacjach będą nie do zaakceptowania.

Obiekt zapisujący dane do strumienia wyjściowego, czyli do pliku będzie wspólny dla wszystkich struktur i będzie zawierał między innymi metodę `WriteEntities`.

```csharp
    internal sealed class BusinessEntityProcess
    {

        ...

        private void WriteEntities(string filePath, IEnumerable entities)
        {
            using (FileStream fs = File.Create(filePath))
            {
                foreach (object entity in entities)
                {
                    byte[] buffer = MarshalHelper.ToByteArray(entity);
                    fs.Write(buffer, 0, buffer.Length);
                }
            }
        }

        ...

    }
```

Metoda `MarshalHelper.ToByteArray` korzysta z klasy `Marshal` aby miejsce w pamięci zajmowane przez dowolny obiekt zapisać do bufora bajtów. Jak widzimy, do metody `WriteEntities` przekazujemy obiekt implementujący interfejs `IEnumerable`. W .NET 1.1 obiekt tworzący odpowiednią kolekcję, która standardowo implementuje wymagany interfejs, mógł wyglądać następująco:

```csharp
    internal sealed class ProductCategoryData : IProductCategoryData
    {
        private const string SqlSelect =
            "SELECT Id, Skrot, Nazwa FROM P_Kategoria";

        #region IProductCategoryData Members

        public ArrayList GetProductCategories(Database database)
        {
            ArrayList list = new ArrayList();

            using (DbDataReader reader = database.ExecuteReader(SqlSelect, null))
            {
                while (reader.Read())
                {
                    ProductCategory productCategory = new ProductCategory();
                    productCategory.Id = reader.GetInt32(0);
                    productCategory.Code = MarshalHelper.EncodeToByteArray(
                    reader.GetString(1), ProductCategory.CodeSize);
                    productCategory.Name = MarshalHelper.EncodeToByteArray(
                    reader.GetString(2), ProductCategory.NameSize);
                    list.Add(productCategory);
                }
            }

            return list;
        }

        #endregion
    }
```

Obiekt `Database` zapewnia nam obsługę połączenia z bazą danych a metoda `MarshalHelper.EncodeToByteArray` dokonuje konwersji pomiędzy łańcuchem Unicode a dowolną stroną kodową. Zmienna `database` umożliwia nam wywołanie polecenia zwracającego nam z bazy SQL strumień danych typu `DbDataReader`. Pamięć dla tych elementów została już przydzielona. Wykonanie powyższej metody zwróci nam wymagany przez metodę `WriteEntities` interfejs `IEnumerable`. Ale skutkować to będzie zajęciem pamięci tym razem dla elementów listy. Zmienna `reader` jest zamykana, co nie oznacza, iż pamięć jest czyszczona. Przy dużych ilościach danych lub przy małej ilości pamięci (urządzenia mobilne) może mieć to spore znaczenie.

Wprowadzone w .NET 2.0 iteratory diametralnie tę sytuację zmieniają. Poniżej klasy implementujące nowe możliwości języka C#:

```csharp
    internal sealed class BusinessEntityProcess
    {

        ...

        private void WriteEntities<T>(string filePath, IEnumerable<T> entities)
        {
            using (FileStream fs = File.Create(filePath))
            {
                foreach (T entity in entities)
                {
                    byte[] buffer = MarshalHelper.ToByteArray(entity);
                    fs.Write(buffer, 0, buffer.Length);
                }
            }
        }

        ...

    }
```

oraz

```csharp
    internal sealed class ProductCategoryData : IProductCategoryData
    {
        private const string SqlSelect =
            "SELECT Id, Skrot, Nazwa FROM P_Kategoria";

        #region IProductCategoryData Members

        public IEnumerable<ProductCategory> GetProductCategories(Database database)
        {
            using (DbDataReader reader = database.ExecuteReader(SqlSelect, null))
            {
                while (reader.Read())
                {
                    ProductCategory productCategory = new ProductCategory();
                    productCategory.Id = reader.GetInt32(0);
                    productCategory.Code = MarshalHelper.EncodeToByteArray(
                    reader.GetString(1), ProductCategory.CodeSize);
                    productCategory.Name = MarshalHelper.EncodeToByteArray(
                    reader.GetString(2), ProductCategory.NameSize);
                    yield return productCategory;
                }
            }
        }

        #endregion
    }
```

Cała tajemnica tkwi w słowie kluczowym `yield`. Powoduje ono przerwanie chwilowe wykonania `while` i zwrócenie sterowania do pętli `foreach`. Dzięki takiemu podejściu unikamy ciągłego przydzielania i zwalniania pamięci dla niedużych elementów kolekcji.

Jednocześnie zostały wprowadzone typy generic. Pętla `foreach` domyślnie rzutuje wszystkie elementy kolekcji na object. Wprowadzenie więc typów generic spowoduje, iż żadnego rzutowania nie będzie, dzięki czemu nasz kod powinien zaoszczędzić jeszcze kilka dodatkowych taktów zegara.

PS. Wielkie dzięki dla wszystkich, którzy komentowali tę notatkę przed odtworzeniem jej po Wielkim Padzie Systemu.