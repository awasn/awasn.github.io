---
title: Wartość domyślna Enum
date: 2008-11-18T22:39:00+01:00
languageCode: pl-pl
description: Wartość domyślna typu wyliczanego Enum w .NET może być poza zakresem
categories:
  - Development
tags:
  - .NET
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2008/11/18/warto-domy-lna-enum.aspx
disqus_disable: true
---

Typy wyliczane umożliwiają zdefiniowanie dozwolonych wartości, które może przyjmować zmienna, parametr metody etc. w trakcie wykonania programu. Każdy ze zdefiniowanych elementów posiada odpowiadającą mu wartość liczbową typu całkowitego. Pierwszy element domyślnie przyjmuje wartość zero. Oczywiście możemy zmienić nie tylko wartość domyślną pierwszego elementu, ale również wszystkich poszczególnych elementów typu. Poniżej prosty przykład, w którym wyliczanie rozpoczynamy od liczby trzy:

```csharp
    internal enum DocumentType : byte
    {
        Receipt = 3,
        Invoice,
        DeliveryNote,
        CustomerPayment,
        VendorPayment,
        PurchaseOrder
    }
```

Co jednak się stanie w sytuacji jeśli nie zainicjujemy zmiennej typu wyliczanego? W dokumentacji napisane jest: "The default value of an enum E is the value produced by the expression (E)0". Cóż to oznacza? Otóż w przypadku braku zainicjowania zmiennej zostanie jest przypisana wartość domyślna typu liczbowego czyli zero! To zaś oznacza, iż w takiej sytuacji jesteśmy poza zakresem. Morał z tego taki, iż warto jednak zaczynać wyliczanie zawsze posiadając wartość domyślną.

```csharp
    internal enum DocumentType : byte
    {
        None,
        Receipt = 3,
        Invoice,
        DeliveryNote,
        CustomerPayment,
        VendorPayment,
        PurchaseOrder
    }
```

Czyli na przykład tak jak powyżej.