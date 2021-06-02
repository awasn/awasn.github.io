---
title: Range<T>
date: 2008-07-11T15:30:00+02:00
lastmod: 2018-06-08T16:58:00+02:00
languageCode: pl-pl
description: Własna implementacja klasy do porównań Range w .NET C#
categories:
  - Development
tags:
  - .NET
  - Design patterns
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/07/11/range-t.aspx
disqus_disable: true
---

Porównań i walidacji w kodzie zawsze dużo jest. Czasem, jak u mnie ostatnio, warto spreparować sobie specjalną klasę operacje tego typu ułatwiającą. Koncept nie jest nowy. Poczytać można o nim między innymi na stronie [Martina Fowlera](http://martinfowler.com/eaaDev/Range.html). Ciekawa natomiast jest implementacja, którą można wykonać korzystając z platformy .NET.

Pierwsza wersja klasy w najważniejszych swoich częściach wyglądała następująco:

```csharp
    class Range<T>
    {
        private readonly T minimum;
        private readonly T maximum;
        private readonly bool rangeIsExclusive;

        public bool Included(T value)
        {
            Debug.Assert(value != null);

            if(minimum > value){
                return false;
            }
            if(minimum == value && rangeIsExclusive){
                return false;
            }
            if (maximum < value) {
                return false;
            }
            if (maximum == value && rangeIsExclusive) {
                return false;
            }

            return true;
        }
    }
```

Nie było to najszczęśliwsze rozwiązanie. Skąd kompilator ma wiedzieć jak są dla potencjalnego typu zakresu zdefiniowane operacje porównania? Dla liczb całkowitych czy rzeczywistych jest to oczywiste. Ale dla obiektów zawierających np. imię i nazwisko pracownika już nie.

Jak zatem podejść do tematu? Rozwiązania należy szukać wśród interfejsów. Nakładamy na typ zakresu naszej klasy więzy, iż musi on implementować interfejs `IComparable<T>` umożliwiający porównywanie klas i struktur. Pełny kod źródłowy poniżej.

```csharp
    /// <summary>
    /// Klasa opisująca dowolny zakres.
    /// </summary>
    /// <typeparam name="T">Typ wybranego zakresu.</typeparam>
    internal sealed class Range<T> where T : IComparable<T>
    {
        private readonly T minimum;
        private readonly T maximum;
        private readonly bool rangeIsExclusive;

        /// <summary>
        /// Konstruktor.
        /// </summary>
        /// <param name="minimum">Dozwolona wartość minimalna.</param>
        /// <param name="maximum">Dozwolona wartość maksymalna.</param>
        /// <remarks>
        /// Domyślnie badana wartość może być równa wartości minimalnej
        /// lub maksymalnej.
        /// </remarks>
        public Range(T minimum, T maximum)
            : this(minimum, maximum, false)
        {
        }

        /// <summary>
        /// Konstruktor.
        /// </summary>
        /// <param name="minimum">Dozwolona wartość minimalna.</param>
        /// <param name="maximum">Dozwolona wartość maksymalna.</param>
        /// <param name="rangeIsExclusive">Czy badana wartość musi być
        /// różna od wartości minimalnej i maksymalnej (nierówność
        /// ostra).</param>
        public Range(T minimum, T maximum, bool rangeIsExclusive)
        {
            Debug.Assert(minimum != null);
            Debug.Assert(maximum != null);
            Debug.Assert(minimum.CompareTo(maximum) <= 0);

            this.minimum = minimum;
            this.maximum = maximum;
            this.rangeIsExclusive = rangeIsExclusive;
        }

        /// <summary>
        /// Dozwolona wartość minimalna.
        /// </summary>
        public T Mimimum
        {
            get { return minimum; }
        }

        /// <summary>
        /// Dozwolona wartość maksymalna.
        /// </summary>
        public T Maximum
        {
            get { return maximum; }
        }

        /// <summary>
        /// Czy podawana parametrem wartość mieści się w zadanym zakresie.
        /// </summary>
        /// <param name="value">Wartość, którą chcemy sprawdzić.</param>
        /// <returns>Metoda zwraca <b>true</b> jeśli podana wartość mieści
        /// się w zakresie.</returns>
        public bool Included(T value)
        {
            Debug.Assert(value != null);

            int result;

            result = minimum.CompareTo(value);
            if((result > 0) || (result == 0 && rangeIsExclusive)) {
                return false;
            }

            result = maximum.CompareTo(value);
            if((result < 0) || (result == 0 && rangeIsExclusive)) {
                return false;
            }

            return true;
        }
    }
```

Przykładowe wykorzystanie:

```csharp
            Range<double> discountRange = new Range<double>(0, 99.99);
            return discountRange.Included(23);
```

I to wszystko. Od teraz życia powinno stać się łatwiejsze. Przynajmniej w temacie porównywania zakresów.

Mała aktualizacja kilka godzin po publikacji posta:

* Wojtek Gębczyk zwrócił mi uwagę, iż nazwa f-cji oznacza prędzej dodanie do zbioru niż sprawdzenie bycia w danym zakresie. I miał rację. Dlatego też zmieniłem nazwę metody `Include` na `Included`. Dzięki Wojtek;
* Michał Grzegorzewski zaproponował sprawdzanie czy podana wartość maksymalna jest większa równa od wartości minimalnej. Dodałem `Assert`. Dzięki Michał;
* I mała dyskusja jak nazwać inaczej metodę `Included`.... Może `Contains`? Ktoś ma jakieś pomysły?