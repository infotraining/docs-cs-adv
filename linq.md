# LINQ
## Podstawy LINQ

**Language INtegrated Query (LINQ)** – mechanizm umożliwiający pisanie zapytań dotyczących lokalnych kolekcji obiektów oraz zdalnych źródeł danych przy jednoczesnym zachowaniu silnej kontroli typów. Dostarcza zunifikowany sposób wykonywania zapytań niezależnie od rodzaju źródeł danych. Upraszcza tworzenie aplikacji, które zarządzają danymi.
![image](_images/LINQ-structure.*)

### Podstawowe pojęcia LINQ

Zapytanie LINQ jest wyrażeniem, które umożliwia transformację sekwencji za pomocą operatora zapytania.

* **Sekwencja** - dowolny obiekt implementujący generyczny interfejs `IEnumarable<T>`
* **Element** – pojedynczy element sekwencji
* **Operator zapytania** – metoda przekształcająca sekwencję. Typowy operator akceptuje sekwencję wejściową i generuje przetransformowaną sekwencję wyjściową. 

Wyrażenia LINQ mogą być tworzone na dwa sposoby:
1.  Przy pomocy składni zapytań LINQ
2.  Przy pomocy operatorów zapytania i wyrażeń lambda

#### Proste zapytanie LINQ

Kolekcja źródłowa:

```csharp
string[] names = { "Ala", "Ola", "Marek", "Zenon" };

```

Składnia zapytań:

```csharp
IEnumerable<string> subset = from n in names
                             where n.Length == 3
                             select n;

```

Operator zapytania + wyrażenia lambda:

```csharp
IEnumerable<string> theSameSubset = 
    names.Where(name => name.Length == 3).Select(name => name);

```

## Operatory zapytania LINQ

Operatory zapytania zdefiniowane są jako metody rozszerzające w klasie `Enumerable` znajdującej się w przestrzeni nazw `System.Linq`.
Podstawowe operatory zapytania:

* operatory filtrujące: `Where`, `Take`, `Skip`, `TakeWhile`, `SkipWhile`, `Distinct`
* operatory projekcji: `Select`, `SelectMany`   
* operatory elementowe: `First`, `FirstOrDefault`, `Last`, `LastOrDefault`, `Single`, `SingleOrDefault`, `ElementAt`, `ElementAtOrDefault`, `DefaultIfEmpty`
* operatory agregacji: `Count`, `Min`, `Max`, `Sum`, `Average`, `Aggregate`
* kwantyfikatory: `Contains`, `Any`, `All`, `SequenceEquals`
* operatory zbiorów: `Concat`, `Union`, `Intersect`, `Except` 
* operatory konwersji: 

Operatory zapytania często przyjmują jako parametry obiekty typu `Func<TSource, TResult>` (najczęściej wyrażenia lambda).

### Operatory filtrujące

Operator `Where` umożliwia filtrowanie sekwencji wejściowej.

```csharp
static IEnumerable<TSource> Where<TSource>(
    this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
    foreach(TSource element in source)
        if (predicate(element))
            yield return element;
}

```

Metoda Where umieszcza w sekwencji wyjściowej wszystkie elementy sekwencji wejściowej spełniające predykat

* `source` - reprezentuje sekwencję wejściową
* `predicate` - delegat wywoływany dla każdego elementu sekwencji wejściowej

Operator `OfType` umożliwia filtrowanie elementów danego typu.

```csharp
System.Collections.ArrayList fruits = new System.Collections.ArrayList(4);
fruits.Add("Mango");
fruits.Add("Orange");
fruits.Add("Apple");
fruits.Add(3.0);
fruits.Add("Banana");

// Apply OfType() to the ArrayList.
IEnumerable<string> q1 = fruits.OfType<string>();

```

### Operator projekcji

#### Operator Select

Fundamentalnym operatorem zapytania jest metoda `Select`, która przekształca (dokonuje projekcji) każdy element sekwencji wejściowej, za pomocą podanego wyrażenia lambda.

```csharp
    string[] names = { "Ala", "Ola", "Marek", "Zenon" };

    IEnumerable<string> upperNames = names.Select(name => name.ToUpper());

```

```console
"ALA", "OLA", "MAREK", "ZENON"

```

Zapytaniem `Select` można dokonywać projekcji na typ anonimowy.

```csharp
var namesWithLength = names.Select(name => new { Name=name, Length=name.Length });

```

```csharp
{ Name = Ala, Length = 3 } { Name = Ola, Length = 3 }
{ Name = Marek, Length = 5 } { Name = Zenon, Length = 5 }

```

Wyrażenie lambda, które dokonuje projekcji, może akceptować jako drugi argument wartość całkowitą pełniącą rolę indeksu (pozycji elementu w kolekcji źródłowej).

```csharp
string[] names = { "Ala", "Ola", "Marek", "Zenon" };

IEnumerable<string> indexedNames = names.Select((name, i) => string.Format("[{0} - {1}]", i, name);

```

```console
"[0 - Ala]", "[1 - Ola]", "[2 - Marek]", "[3 - Zenon]"

```

#### Operator SelectMany

`SelectMany` umożliwia spłaszczenie danych.
Wyrażenie lambda przekazane jako argument musi zwracać jako wynik `IEnumerable`. W rezultacie każdy obiekt sekwencji wejściowej jest transformowany przez funkcję zwracającą sekwencję. Końcowy rezultat jest konkatenację wielu sekwencji do jednej sekwencji wynikowej.

```csharp
string[] fullNames = { "Anne Williams", "John Fred Smith", "Sue Green" };

IEnumerable<string> query = fullNames.SelectMany (name => name.Split());

```

```console
"Anne", "Williams", "John", "Fred", "Smith", "Sue", Green"

```

### Operatory elementowe

Operatory elementowe zwracają pojedyncze elementy zamiast kolekcji:

* `First` – pierwszy element.
* `Last` – ostatni element.
* `Single` – zwraca pojedynczy element, jeśli kolekcja zawiera więcej niż jeden element, rzucany jest wyjątek InvalidOperationException.
* `ElementAt` – zwraca element znajdujący się na wskazanej pozycji.

```csharp
int[] numbers = { 1, 3, 6, 5, 8, 7, 10 };
int firstNumber = numbers.First(); // 1
int lastNumber = numbers.Last(); // 10
int thirdNumber = numbers.ElementAt(2); // 6
int firstEvenElement = numbers.First(n => n % 2 == 0); // 6

```

`FirstOrDefault`, `LastOrDefault`, `SingleOrDefault`, `ElementOrDefault` zwracają element albo wartość domyślną (dla typu referencyjnego `null`).
### Partycjonowanie danych

Partycjonowanie w LINQ polega na podziale sekwencji na dwie części, bez reorganizacji elementów, i zwróceniu jednej z tych części.

* `Skip` - pomija elementy aż do podanej pozycji w sekwencji
* `SkipWhile` - pomija elementy spełniające podany predykat, aż do pierwszego elementu, który go nie spełnia
* `Take` - pobiera elementy aż do podanej pozycji w sekwencji
* `TakeWhile` - pobiera elementy spełniające predykat, aż do pierwszego elementu, który go nie spełnia

```csharp
int[] grades = { 59, 82, 70, 56, 92, 98, 85 };

IEnumerable<int> lowerGrades =
    grades.OrderByDescending(g => g).Skip(3);

```

```console
82 70 59 56

```

### Operatory porządkujące

Operatory porządkujące określają kolejność elementów w sekwencji wyjściowej:

* `OrderBy`, `ThanBy` – zwraca sekwencję z elementami uporządkowanymi rosnąco.
* `OrderByDescending`, `ThanByDescending` – zwraca sekwencję z elementami uporządkowanymi malejąco.
* `Reverse` – zwraca sekwencję z elementami uporządkowanymi odwrotnie niż w sekwencji wejściowej.

```csharp
string[] names = { "Ala", "Ola", "Marek", "Zenon", "Ewa", "Euzebiusz", "Leszek" };

IEnumerable<string> sortedNames = names.OrderBy(n => n.Length).ThenBy(n => n);

```

```console
Ala Ewa Ola Marek Zenon Leszek Euzebiusz

```

### Operatory agregacji

Operatory agregacji zwracają wartość skalarną (najczęściej liczbową):

* `Count` – zwraca liczbę elementów. Przyjmuje opcjonalny predykat, który decyduje o zaliczeniu lub pominięciu danego elementu przy zliczaniu.
* `Min` – zwraca wartość najmniejszego elementu.
* `Max` – zwraca wartość największego elementu.
* `Average` – zwraca średnią wartości elementów.
* `Aggregate` - pozwala na zaawansowaną agregację, z użyciem własnego algorytmu agregującego (nie jest wspierany przez LINQ to SQL oraz LINQ to Entities)

```csharp
int[] numbers = { 10, 9, 8, 7, 6 };
int count = numbers.Count();  // 4
int min = numbers.Min();  // 6
int max = numbers.Max();  // 10
int avg = numbers.Average();  // 8

```

`Min`, `Max` i `Average` mogą przyjmować opcjonalny argument określający przekształcenie wartości przed dokonaniem jego agregacji.

```csharp
int evenNumbers = numbers.Count(n => n % 2 == 0);  // 3
int maxReminderAfterDivBy5 = numbers.Max(n => n % 5);  // 4
double rms = Math.Sqrt(numbers.Average(n => n * n)); 

```

`Aggregate` posiada dwie wersje:
1. agregacja z początkową wartością

```csharp
   int[] numbers = { 1, 2, 3 };

   int result1 = numbers.Aggregate(0, (total, n) => total * n); // 0 * 1 * 2 * 3 = 0

```

2. agregacja bez początkowej wartości

```csharp
   int result2 = numbers.Aggregate((total, n) => total); // 1 * 2 * 3 = 6

```

### Kwantyfikatory

Kwantyfikatory zwracają dla przekazanej sekwencji wartość logiczną `bool`:

* `Contains`
* `Any`
* `All`
* `SequenceEquals`

```csharp
int[] numbers = { 10, 9, 8, 7, 6 };
bool hasTheNumberNine = numbers.Contains(9);  // true
bool hasMoreThanZeroElements = numbers.Any();  // true
bool hasOddNumber = numbers.Any(n => n % 2 == 1);  // true
bool allOddsNumbers = numbers.All(n => n % 2 == 1);  // false

```

### Operatory zbiorów

Operatory zbiorów operują na parze sekwencji tego samego typu:

* `Concat` – dodaje jedną sekwencję do drugiej
* `Union` – dodaje jedną sekwencję do drugiej, usuwając duplikaty
* `Intersect` – część wspólna sekwencji
* `Except` – różnica 

```csharp
int[] seq1 = { 1, 2, 3 };  int[] seq2 = { 3, 4, 5 };
IEnumerable<int> 
    concat = seq1.Concat(seq2),  // { 1, 2, 3, 3, 4, 5 }
    union = seq1.Union(seq2),    // { 1, 2, 3, 4, 5 }
    commonality = seq1.Intersect(seq2),  // { 3 }
    diff1 = seq1.Except(seq2),   // { 1, 2 }
    diff2 = seq2.Except(seq1);   // { 4, 5 }

```

### Opóźnione wykonanie

Operatory zapytania realizują tzw. opóźnione wykonanie (*deferred execution*). Są wykonywane nie w momencie konstrukcji, ale dopiero w momencie przeliczania sekwencji wynikowej pętlą `foreach`.

```csharp
List<int> numbers = new List<int> { 1 };
numbers.Add(2);

IEnumerable<int> numSequence = numbers.Select(n => n * 10);

numbers.Add(3);

foreach (int n in numSequence)
    Console.Write(n + " | ");

```

```console
10 | 20 | 30

```

Opóźnione wykonanie dotyczy wszystkich operatorów standardowych z wyjątkiem:

* operatorów zwracających pojedynczy element albo wartość skalarną (operatorów elementowych, agregacji i kwantyfikatorów)
* operatorów konwersji: `ToArray`, `ToList`, `ToDictionary`, `ToLookup`

Opóźnione wykonanie powoduje, że zapytanie jest ponownie ewaluowane przy kolejnej enumeracji.
### Operatory konwersji

Operatory konwersji  `ToArray`, `ToList`, `ToDictionary`, `ToLookup` umożliwiają buforowanie wyników operacji z konkretnego momentu w czasie oraz uniknięcie ponownego wykonania zasobożernego zapytania lub zapytania odwołującego się do zewnętrznych źródeł danych.

```csharp
int[] numbers = { 1, 2, 5, 8, 10 };

List<int> numbersTimesTen = 
    numbers
        .Select(n => n * 10)
        .ToList();  // operator wykonywany natychmiast

```

### Kaskadowe wywołania

Do budowania bardziej złożonych zapytań można wykorzystać kaskadowe wywołania pojedynczych operatorów zapytań.
W trakcie kaskadowych wywołań operatorów działa również opóźnione wywołanie zapytań.

```csharp
string[] names = {"Ala","Ola","Marek","Zenon","Ewa","Euzebiusz","Leszek"};

IEnumerable<string> query = names
    .Where( n => n.Contains("e"))
    .OrderBy( n=> n.Length)
    .ThenBy( n => n)
    .Select( n => n.ToUpper());

```

```console
MAREK ZENON LESZEK EUZEBIUSZ

```

### Operacje generujące sekwencje wartości

Klasa `Enumerable` implementuje operacje, które pozwalają tworzyć sekwencje zawierające odpowiednie 
wartości.

* `Range`  - generuje kolekcję zawierającą sekwencję wartości
* `Repeat` - generuje kolekcję, która zawiera powtarzającą się wartość

```csharp
IEnumerable<int> squares = Enumerable.Range(1, 10).Select(x => x * x);

IEnumerable<string> sameValues = Enumerable.Repeat("same", 15);

```

### Zipping sekwencji

Operator `Zip` umożliwia scalenie parami dwóch sekwencji. Scalane są elementy mające te same indeksy 
w obu sekwencjach.

```csharp
int[] numbers = { 1, 2, 3, 4 };
string[] words = { "one", "two", "three" };

var numbersAndWords = numbers.Zip(words, (first, second) => first + " " + second);

```

```console
"1 one", "2 two", "3 three"

```

## Składnia zapytań

Operatory zapytań + lambda:

```csharp
IEnumerable<string> query = 
    names
        .Where( n => n.Contains("e"))
        .OrderBy( n=> n.Length)
        .ThenBy( n => n)
        .Select( n => n.ToUpper());

```

Składnia zapytań:

```csharp
IEnumerable<string> query = 
    from n in names
    where n.Contains("e")
    orderby n.Length, n
    select n.ToUpper();

```

Kompilator przetwarza zapytanie ujęte w kodzie tłumacząc je na wywołania odpowiednich operatorów i  wyrażeń lambda. 
Składnia zapytań ułatwia pisanie zapytań LINQ.
![image](_images/linq-query-syntax.*)

Dotyczy jedynie niewielkiego podzbioru operatorów zapytań:

*  `Where`
*  `Select`
*  `SelectMany`
*  `OrderBy`
*  `ThanBy`
*  `OrderByDescendng`
*  `ThenByDescending`
*  `Group`
*  `Join`
*  `GroupJoin`

W przypadku zapytań stosujących inne operatory konieczne jest:

* zapisanie zapytania w całości z wykorzystaniem operatorów i wyrażeń lambda
* skonstruowanie mieszanego zapytania wykorzystującego składnię zapytań oraz operatory zapytań

```csharp
var query = 
    from n in names
    where n.Length == names.Min(n2 => n2.Length)
    select n;

```

### Słowo kluczowe ``let``

Słowo kluczowe `let` umożliwia wprowadzenie nowej zmiennej obok zmiennej iteracji. Pozwala na realizowanie obliczeń dotyczących każdego elementu sekwencji wejściowej bez tracenia wartości pierwotnej tego elementu.

```csharp
string[] names = { "Marek", "Ala", "Ola", "Gosia", "Kamil", "Jerzy" };

IEnumerable<string> query =
    from n in names
    let voweless = Regex.Replace(n, "[aąeęioóuy]", "")
    where voweless.Length > 2
    select n + " - " + voweless;

```

```console
Marek - Mrk
Kamil - Kml
Jerzy - Jrz

```

### Kontynuacja zapytań

Słowo kluczowe `into` umożliwia kontynuowanie zapytania po klauzulach `select` lub `group`.

```csharp
IEnumerable<string> query =
    from n in "And now something completely different".Split()
    select n.ToUpper()
        into upper
        where upper.Length > 5
        select upper;

```

```console
SOMETHING COMPLETELY DIFFERENT

```

### Zapytania z wieloma generatorami

Zapytanie LINQ może zawierać więcej niż jeden generator – więcej niż jedną klauzulę `from`. Wynikiem jest iloczyn kartezjański sekwencji wejściowych.
W celu obsługi zapytania kompilator niejawnie wywołuje operator `SelectMany`:

```csharp
int[] numbers = { 1, 2, 3 };
string[] words = { "a", "b" };

IEnumerable<string> query =
    from n in numbers
    from w in words
    select n.ToString() + " " + w;

```

```console
{ "1 a", "1 b", "2 a", "2 b", "3 a", "3 b" }

```

## Złączenia, porządkowanie i grupowanie

LINQ udostępnia operatory złączeń do wykonywania złączeń zbiorów danych na bazie dopasowania wartości – kluczy.
Operatory złączeń:

* obsługują jedynie złączenia równościowe (equi-join)
* `Join` – zwraca płaski zbiór wynikowy (*inner join*)
* `GroupJoin` – zwraca w wyniku zbiór hierarchiczny, pogrupowany według elementów zewnętrznego zbioru źródłowego (*inner join*, *left outer join*)
### Złączenia płaskie

Składnia:

```csharp
from zmienna-iteracji-sekw-zewn in sekw-zewn
join zmienna-iteracji-sekw-wewn in sekw-wewn
    on wyr-klucza-zlaczenia-zewn equals wyr-klucza-zlaczenie-wewn 

```

Przykład:

```csharp
var customers = new[] {
    new { ID = 1, Name = "IBM" },
    new { ID = 2, Name = "Thomson-Reuters" }, 
    new { ID = 3, Name = "Sabre Holdings" }
};

var purchases = new[] {
    new { CustomerID = 1, Product = "Laptop HP 6930p" },
    new { CustomerID = 2, Product = "Drukarka Epson R800" },
    new { CustomerID = 2, Product = "Szkolenie .NET Framework 3.5" },
    new { CustomerID = 3, Product = "Router Linksys WL5450GL" }
};

IEnumerable<string> query = 
    customers
        .Join(purchases, c => c.ID, p => p.CustomerID, 
              (c, p) => c.Name + " purchased " + p.Product);

```

### Złączenia z grupowaniem

`GroupJoin` – zwraca w wyniku zbiór hierarchiczny, pogrupowany według elementów zewnętrznego zbioru źródłowego.

```csharp
var groupedQuery =
    from c in customers 
    join p in purchases on c.ID equals p.CustomerID
    into customersPurchases
        select new { Name = c.Name, customersPurchases };

```

    var groupedQueryAlt = customers.GroupJoin(purchases, c => c.ID, p => p.CustomerID
                (c, p) => new {Name = c.Name, customersPurchases = p});
### Grupowanie

Operator `GroupBy` – organizuje płaską sekwencję wejściową w sekwencję grup elementów. Odczytuje elementy sekwencji wejściowej do tymczasowego słownika list, w których wszystkie elementy o tej samej wartości kryterium grupowania umieszczane są pod tym samym kluczem, w tej samej liście. Zwraca obiekt implementujący interfejs `IGrouping<T>`

```csharp
public interface IGrouping<TKey, TElement> : IEnumerable<T>, IEnumerable
{
    TKey Key { get; }  // właściwość – klucz grupowania
}

```

Przykład:

```csharp
string[] names = { Ola", "Ala", "Adam", "Ewa", "Zenon", 
        "Marek", "Beata", "Zenobiusz" };

var queryAlt = names.GroupBy(n => n.Length); // składnia operatorów

// składnia zapytań
IEnumerable<IGrouping<int, string>> query =
    from name in names
    group name by name.Length;

foreach (IGrouping<int, string> grouping in query)
{
   Console.Write("Length: {0}  Names:", grouping.Key);

   foreach (string name in grouping)
       Console.Write(" {0}", name);

   Console.WriteLine();
}

```

```console
Length: 3  Names: Ola Ala Ewa
Length: 4  Names: Adam
Length: 5  Names: Zenon Marek Beata
Length: 9  Names: Zenobiusz

```

Grupy zwracane przez operator `GroupBy` nie są posortowane wg wartości klucza. Operator nie realizuje operacji porządkowania. Aby je posortować należy uzupełnić zapytanie o wywołanie operatora `OrderBy`.

```csharp
string[] names = { "Ola", "Ala", "Adam", "Ewa", "Zenon", 
                   "Marek", "Beata", "Zenobiusz" }
var query = 
    from name in names
    group name.ToUpper() by name.Length
    into grouping
    orderby grouping.Key
    select grouping;

```

W zapytaniach LINQ z grupowaniem często wykorzystywana jest kontynuacja zapytań. Na przykład, klauzula `where` działa wówczas jak klauzula `HAVING` w SQL.

```csharp
var query = 
    from name in names
    group name.ToUpper() by name.Length
    into grouping
    where grouping.Count() == 3
    orderby grouping.Key
    select grouping;    

```

```console
{ 
    { Key = 3, {"Ola", "Ala", "Ewa" } },
    { Key = 5, { "Zenon", "Marek", "Beata" } }
}

```

#### Płaskie złączenie zewnętrzne

Aby otrzymać płaskie złączenie zewnętrzne (*flat outer join*) należy, najpierw wykonać `GroupJoin()` (otrzymamy hierarchiczny rezultat *left outer join*), następnie `DefaultIfEmpty()` na każdej sekwencji wynikowej i ostatecznie `SelectMany`.

```csharp
var flatLeftOuterJoin = from c in customers

```

## Architektura LINQ

LINQ dostarcza dwie równoległe architektury:

* lokalne zapytania dla kolekcji obiektów (*LINQ to Objects*)

  - operatory zapytań są zaimplementowane w klasie `Enumerable`
  - delegaty typu `Func<..., TResult>`, które są przyjmowane jako argumenty operatorów są lokalnymi funkcjami IL

```csharp
  public static IEnumerable<TSource> 
      Where<TSource> (this IEnumerable<TSource> source, Func<TSource,bool> predicate)

```

* interpretowane zapytania dla zdalnych źródeł danych

  - zapytania operują na sekwencjach, które implementują interfejs `IQueryable<T>`
  - operatory zapytań są zaimplementowane w klasie `Queryable`, która emituje drzewa wyrażeń (**expression trees**), które są interpretowane w czase wykonania zapytania
  - argumenty operatorów są typu `Expression<Func<..., TResult>`

```csharp
  public static IQueryable<TSource> 
      Where<TSource> (this IQueryable<TSource> source, Expression<Func<TSource,bool>> predicate)

```

![image](_images/LINQ-architecture.*)

### Drzewa wyrażeń

Zapytania do zewnętrznych źródeł danych korzystają z drzew wyrażeń. Drzewa wyrażeń odzwierciedlają strukturę zapytania (np. SQL) i umożliwiają generację zapytania SQL w momencie rozpoczęcia iteracji po sekwencji wynikowej.
Kiedy przypisujemy wyrażenie lambda do zmiennej typu `Expression<Func<...>>` kompilator C# emituje kod, który tworzy drzewo wyrażeń. Elementami drzewa wyrażeń są instancje klas dziedziczących po niegenerycznej klasie bazowej `Expression`. 
Generyczny typ `Expression<T>` odpowiada "typizowanemu wyrażeniu lambda". Dziedziczy po niegenerycznej klasie `LambdaExpression`.
Wyrażenie:

```csharp
Expression<Func<string, bool>> expression = s => s.Length < 5;

```

Jest kompilowane do następującego drzewa wyrażeń:
![image](_images/expression-tree.*)

Kod użytkownika dający identyczny rezultat wyglądałby następująco:

```csharp
ParameterExpression p = Expression.Parameter(typeof(string), "s");
MemberExpression stringLength = Expression.Property(p, "Length");
ConstantExpression five = Expression.Constant(5);

BinaryExpression comparison = Expression.LessThan(stringLength, five);

Expression<Func<string, bool>> lambda = Expression.Lambda<Func<string, bool>>(comparison, p);

```

## LINQ to XML

LINQ to XML wprowadza dużo łatwiejszą obsługę formatu XML niż w przypadku starszych API. Jest lekką fasadą korzystającą z niskopoziomowych klas `XmlReader` i `XmlWriter`.
Zawiera dwa elementy:

* XML DOM - tzw. X-DOM
* operatory zapytań

Umożliwia:

* Tworzenie dokumentów XML
* Parsowanie i filtrowanie dokumentów XML
* Modyfikowanie dokumentów XML
* Zapis dokumentów XML do plików i do strumieni
### X-DOM

Dwie najważniejsze i najczęściej używane klasy w X-DOM to `XElement` oraz `XAttribute`.
![image](_images/x-dom.*)

Klasą bazową jest `XObject`. Definiuje odwołanie do korzenia hierarchii (`Parent`) i opcjonalnie do `XDocument`. `XElement` i `XDocument` są korzeniami hierarchii.
`XDocument` opakowuje korzeń w postaci `XElement` dodając deklarację `XDeclaration`, instrukcje przetwarzania, itp.
Obiekt `XElement` dodaje właściwości umożliwiające zarządzanie atrybutami oraz udostępnia właściwości `Name` i `Value` (nie ma potrzeby bezpośredniej obsługi węzła `XText`).
### Ładowanie i parsowanie dokumentów XML

Zarówno `XElement` oraz `XDocument` dostarczają statycznych metod do parsowania danych XML:

* `Load` - buduje drzewo X-DOM z pliku, URI, strumieni `Stream`, `TextReader` i `XmlReader`
* `Parse` - buduje drzewo X-DOM ze stringa

```csharp
XDocument xdoc = XDocument.Load("http://infotraining.pl/sitemap.xml");

XElement root = XElement.Load(@"c:\xml\data.xml");

XmlReader reader = XmlReader.Create("myData.xml");
XElement element = XElement.Load(reader);

XElement config = XElement.Parse(
    @"<configuration>
        <client enabled='true'>
          <timeout>30</timeout>
        </client>
      </configuration>");

```

### Zapisywanie dokumentów XML

Wywołanie metody `ToString` dla dowolnego węzła konwertuje go do sformatowanego stringa. 
Wyłączenie formatowania - `ToString(SaveOptions.DisableFormatting)`.
Klasy `XElement` oraz `XDocument` zawierają metodę `Save`, która umożliwia zapisanie drzewa X-DOM do pliku lub strumieni typu `Stream`, `TextWriter` i `XmlWriter`.
### Tworzenie dokumentów XML

Konstrukcja funkcjonalna umożliwia budowę drzewa X-DOM w jednym wyrażeniu:

```csharp
XNamespace ns = "http://example.books.com";
XDocument books = new XDocument(
   new XElement(ns + "bookstore",
      new XElement(ns + "book", new XAttribute("ISBN", isbn),
         new XElement(ns + "title", title),
         new XElement(ns + "author", 
            new XElement(ns + "first-name", author.FirstName),
            new XElement(ns + "last-name", author.LastName)
        )
     )
  )
);

books.Save(@"C:\Books.xml");

```

Dzięki niej można budować X-DOM jako rezultat zapytania LINQ:

```csharp
IEnumerable<XElement> xmlProducts =
    from p in northwindDataSet.Products
    select 
        new XElement("Product", new XAttribute("ProductID", p.ProductID),
            new XElement("ProductName", p.ProductName),
            new XElement("UnitPrice", p.UnitPrice),
            new XElement("QuantityPerUnit", p.QuantityPerUnit)
        );

XElement root = new XElement("NorthwindProducts");
root.Add(xmlProducts);

root.Save("NorthwindProducts.xml");

```

### Nawigacja po drzewie X-DOM
#### Nawigacja po węzłach podrzędnych

| Typ zwracany          | Metoda                                                                                                                                                                                |
|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| XNode                 | FirstNode { get; }<br>LastNode { get; }                                                                                                                                              |
| IEnumerable<XNode>    | Nodes()<br>DescendantNodes()<br>DescendantNodesAndSelf()                                                                                                                             |
| XElement              | Element(XName)                                                                                                                                                                        |
| IEnumerable<XElement> | Elements()<br>Elements(XName)<br>Descendants()<br>Descendants(XName)<br>DescendantsAndSelf()<br>DescendantNodesAndSelf(XName)                                                        |
| bool                  | HasElements { get;}                                                                                                                                                                   |

* `XElement.Elements()` – zwraca węzły bezpośrednio podrzędne w kolejności występowania w dokumencie
* `XElement.Descandants()` – zwraca węzły podrzędne w kolejności występowania w dokumencie

```csharp
XElement root = XElement.Load("NorthwindProducts.xml");

Console.WriteLine("***** Elements *****");
foreach (XElement e in root.Elements())
    Console.WriteLine(e.Element("ProductName").Value);

Console.WriteLine("\n\n***** Descendants *****");
foreach (XElement e in root.Descendants("ProductName"))
    Console.WriteLine(e.Value);

```

#### Nawigacja po węzłach nadrzędnych

| Typ zwracany        | Metoda                                                                           |
|---------------------|----------------------------------------------------------------------------------|
| XElement            | Parent { get; }                                                                  |
| Enumerable<Element> | Ancestors()<br>Ancestors(XName)<br>AncestorsAndSelf()<br>AncestorsAndSelf(XName) |

#### Nawigacja po atrybutach

| Typ zwracany            | Metoda                                                                |
|-------------------------|-----------------------------------------------------------------------|
| bool                    | HasAttribute { get; }                                                 |
| XAttribute              | Attribute(XName)<br>FirstAttribute { get; }<br>LastAttribute { get; } |
| IEnumerable<XAttribute> | Attributes()<br>Attributes(XName)                                     |

### Modyfikowanie dokumentów XML

Drzewo XML reprezentowane przez obiekty XElement jest modyfikowalne.
Modyfikacja jest możliwa z wykorzystaniem takich metod jak:

* `SetValue` lub zmiana wartości za pomocą  właściwości `Value`

```csharp
  XElement settings = new XElement("settings",
                          new XElement("timeout", 30)
                      );
  settings.SetValue("new value");

```

* `SetElementValue(XName name, object value)` lub `SetAttributeValue(XName name, object value)`

```csharp
  XElement settings = new XElement ("settings");
  settings.SetElementValue ("timeout", 30);      // Adds child node
  settings.SetElementValue ("timeout", 60);      // Update it to 60

```

* `RemoveNodes()`, `RemoveAttributes()`, `RemoveAll()`
* `ReplaceNodes(params object[] content)`, `ReplaceAttributes(params object[] content)`, `ReplaceAll(params object[] content)`

Metoda rozszerzająca `Remove` umożliwia usunięcie sekwencji węzłów lub atrybutów:

```csharp
XElement contacts = XElement.Parse(
    @"<contacts>
        <customer name='Mary'/>
        <customer name='Chris' archived='true'/>
        <supplier name='Susan'>
          <phone archived='true'>012345678<!--confidential--></phone>
        </supplier>
      </contacts>");

// removing comments
contacts.DescendantNodes().OfType<XComment>().Remove();

```

## LINQ to DataSet

LINQ to DataSet umożliwia wygodne manipulowanie danymi przechowywanymi w komponencie DataSet za pomocą wyrażeń LINQ. 

* Zestaw: `System.Data.DataSetExtensions.dll`
* Najważniejsze typy:

  - `DataTableExtensions` oraz `DataRowExtensions`
  - `TypedTableBaseExtensions`
Aby przekształcić obiekt typu `DataTable` na obiekt kompatybilny z LINQ należy wywołać metodę rozszerzającą `AsEnumerable()`:

```csharp
DataTable data = ...;

IEnumerable<DataRow> enumData = data.AsEnumerable();
var enumData = data.AsEnumerable();

```

Aby uniknąć rzutowania w wyrażeniach LINQ należy zastosować metodę rozszerzającą Field<T>:

```csharp
var query = from p in dsProducts.Tables["Products"].AsEnumerable()
            select new
            {
                 ProductName = (string)p["ProductName"],
                 UnitPrice = p.Field<decimal>("UnitPrice")
            };

```

Aby zapełnić nowy obiekt DataTable danymi otrzymanymi na podstawie zapytania LINQ należy skorzystać z metody `CopyToTable<T>()`:

```csharp
var expensiveProducts = 
    from p in dsProducts.Tables["Products"].AsEnumerable()
    where p.Field<decimal>("UnitPrice") > 100.0M
    orderby p.Field<decimal>("UnitPrice")
    select p;

DataTable expensive = expensiveProducts.CopyToDataTable<DataRow>();

```
