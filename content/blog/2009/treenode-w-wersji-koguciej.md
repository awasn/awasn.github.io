---
title: TreeNode w wersji koguciej
date: 2009-12-20T17:30:00+01:00
languageCode: pl-pl
description: Własna implementacja struktury hierarchicznej TreeNode w .NET C#
categories:
  - Development
tags:
  - .NET
publishedOn: http://zine.net.pl/blogs/arkadiusz_wasniewski/archive/2009/12/20/treenode-w-wersji-koguciej.aspx
disqus_disable: true
---

Kilkukrotnie już zdarzyło się, iż potrzebowałem klasy, która umożliwiłaby zapamiętanie typowanych (typed, generic) struktur hierarchicznych (hierarchical collection) czyli dowolnego obiektu wraz z jego elementami potomnymi. W ramach platformy .NET istnieją już klasy implementujące podobną funkcjonalność. Mowa tu oczywiście o `TreeNode` z `TreeNodeCollection` oraz o, bardziej hermetycznym, `MenuItem` wraz z wewnętrznym `MenuItemCollection`. Klasy przeznaczone do obsługi menu trudno byłoby użyć do własnych rozwiązań. `TreeNode` jest zaś "ciężka" i brak w niej typowania, czyli możliwości określenia typu przechowywanego obiektu (precz z rzutowaniem!).

Z tego też powodu stworzyłem razu pewnego własną implementację struktury hierarchicznej. Nie jest ona skomplikowana. Punktem wyjścia jest klasa przechowująca dane wraz z kolekcją elementów potomnych:

```csharp
    internal class LightTreeNode<T>
    {
        private readonly T _item;
        private readonly LightTreeNodeCollection<T> _nodes;

        public LightTreeNode(T item)
        {
            _item = item;
            _nodes = new LightTreeNodeCollection<T>(this);
        }

        public T Value
        {
            get { return _item; }
        }

        public LightTreeNodeCollection<T> Nodes
        {
            get { return _nodes; }
        }
    }
```

Kolekcja przechowująca potomków dla ułatwienia operowania implementuje interfejs `ICollection<T>`:

```csharp
    internal class LightTreeNodeCollection<T> : ICollection<LightTreeNode<T>>
    {
        private readonly List<LightTreeNode<T>> _list;
        private readonly LightTreeNode<T> _owner;

        public LightTreeNodeCollection(LightTreeNode<T> owner)
        {
            _owner = owner;
            _list = new List<LightTreeNode<T>>();
        }

        public LightTreeNode<T> Owner
        {
            get { return _owner; }
        }

        public LightTreeNode<T> Add(T item)
        {
            var node = new LightTreeNode<T>(item);
            _list.Add(node);
            return node;
        }

        #region Implementation of IEnumerable

        public IEnumerator<LightTreeNode<T>> GetEnumerator()
        {
            return _list.GetEnumerator();
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }

        #endregion

        #region Implementation of ICollection<LightTreeNode<T>>

        public void Add(LightTreeNode<T> item)
        {
            _list.Add(item);
        }

        public void Clear()
        {
            for(int i = 0; i < _list.Count; i++){
                _list[i].Nodes.Clear();
            }
            _list.Clear();
        }

        public bool Contains(LightTreeNode<T> item)
        {
            return _list.Contains(item);
        }

        public void CopyTo(LightTreeNode<T>[] array, int arrayIndex)
        {
            _list.CopyTo(array, arrayIndex);
        }

        public bool Remove(LightTreeNode<T> item)
        {
            int index = _list.IndexOf(item);
            if(index < 0){
                return false;
            }
            _list[index].Nodes.Clear();
            _list.Remove(item);
            return true;
        }

        public int Count
        {
            get { return _list.Count; }
        }

        public bool IsReadOnly
        {
            get { return false; }
        }

        #endregion
    }
```

Przykład zastosowania to chociażby przygotowanie dla widoku (**View**) poleceń menu z poziomu prezentera (**Presenter**, **Presentation Model**):

```csharp
            var menu = new LightTreeNode<Command>(new Command("Opcje"));
            LightTreeNode<Command> submenu = menu.Nodes.Add(
                new Command("Nawigacja"));
            submenu.Nodes.Add(new Command("Pierwszy", MoveFirst));
            submenu.Nodes.Add(new Command("Ostatni", MoveLast));
```

PS. Dla uproszczenia kodu usunięto wszystkie wywołania `Debug.Assert` sprawdzające poprawność wywołań.