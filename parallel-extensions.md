# Parallel Extensions

**Parallel Extensions** to biblioteka, która została zaprojektowana w celu ułatwienia tworzenia aplikacji opartych na współbieżności i równoległości w językach platformy .NET. 
Biblioteka składa się z następujących elementów:

* **Task Parallel Library (TPL)**, która dostarcza rozwiązań opartych na zadaniach równoległych (imperative data and task parallelism).
* **Parallel LINQ (PLINQ)**, która dostarcza  wsparcia dla tzw. declarative data parallelism.
* **Coordination Data Structures (CDS)**, który dostarcza wsparcia do koordynacji i zarządzania stanem współdzielonych zadań.

Główną koncepcją Parallel Extensions jest zadanie (`Task`), które jest małą jednostką kodu, przeważnie instrukcją lambda, która może być uruchamiana niezależnie od pozostałych fragmentów kodu. Zarówno PLINQ, jak i TPL dostarczają metody służące do tworzenia zadań, przy czym PLINQ oferuje mniej skomplikowany i wymagający model zadaniowy. 
Parallel Extensions zawiera tzw. **Task Manager**, który porządkuje zadania do wykonania (globalna kolejka zadań). Dodatkowo, Task Manager zarządza relacjami typu  zadanie – wątek. Jedno zadanie może być rozbite na kilka wątków lub jeden wątek może obsługiwać wiele zadań, a dzieje się to poza kontrolą programisty. 
Domyślny model zachowania systemu porządkowania zadań kreuje tyle wątków, ile jest procesorów lub rdzeni procesora na komputerze, na którym aplikacja jest uruchomiona. Każdy wątek jest powiązany z osobną kolejką zadań, a dodatkowo, kiedy przechodzi w stan jałowy (*idle*) pobiera pakiet zadań czekających na realizację, przenosi go do lokalnej kolejki i na koniec wykonuje po jednym zadaniu z tej kolejki. System został zoptymalizowany w ten sposób, aby nie doprowadzić do sytuacji, kiedy jeden wątek wykona całą pracę i będzie bezczynnie oczekiwać na zakończenie pracy pozostałych wątków. Wątek, który został przetworzył wszystkie żądania, będzie sprawdzał kolejkę pozostałych wątków i w razie potrzeby pobierał  z niej zadania, które najdłużej oczekują na wykonanie (*task stealing*). 

## PLINQ

LINQ szeroko wykorzystywany w .NET 4.5 do manipulowania kolekcjami obiektów został wzbogacony o możliwość wykonywania tych samych operacji współbieżnie. W praktyce oznacza to, że każda iteracyjna operacja wykonana na kolekcji za pomocą PLINQ zostaje opakowana w zestaw zadań wykonywanych współbieżnie.Przy czym operacja jako całość zostanie ukończona pod warunkiem, że ukończone zostaną wszystkie jej zadania. 
Ten mechanizm ma zastosowanie przy LINQ to Object, ale nie dla LINQ to XML oraz LINQ to SQL. 
Kluczowe metody `Parallel` dla interfejsu `IEnumerable`:

| Metoda                        | Opis                                                                     |
|-------------------------------|--------------------------------------------------------------------------|
| `AsParallel()`                | Konwertuje zapytanie do type `ParallelQuery`                             |
| `AsSequential()`              | Konwertuje `ParallelQuery` do zwykłego zapytania                         |
| `AsOrdered()`                 | Nakazuje zachowanie kolejności zadań                                     |
| `AsUnordered()`               | Znosi nakaz zachowania kolejności zadań                                  |
| `WithCanceletion()`           | Modyfikuje `ParallelQuery` dodając możliwość anulowania zadania          |
| `WithDegreeOfParallelism()`   | Ogranicza maksymalną liczbę zadań zapytaniu                              |
| `WithExecutionMode()`         | Wymusza użycie współbieżności w zapytaniu lub pozostawia decyzję PLINQ   |
| `WithMergeOption()`           | Określa, czy wynik zapytania ma być buforowany                           |

Przykład z ignorowaniem kolejności zadań:

```csharp
var sourceData = Enumerable.Range(1, 10).ToArray();

var results = from item in sourceData.AsParallel().AsUnordered()
              select Math.Pow(item, 2);

foreach (var d in results)
{
    Console.WriteLine("Parallel:{0}", d);
}

```

```console
Parallel: 25
Parallel: 0
Parallel: 36
Parallel: 1
Parallel: 49
Parallel: 4
Parallel: 64
Parallel: 9
Parallel: 81
Parallel: 16
Press enter to finish

```

Przykład z zachowaniem kolejności:

```csharp
var sourceData = Enumerable.Range(1, 10).ToArray();

var results = from item in sourceData.AsParallel().AsOrdered()
              select Math.Pow(item, 2);

foreach (var d in results)
{
    Console.WriteLine("Parallel:{0}", d);
}

```

```console
Parallel: 0
Parallel: 1
Parallel: 4
Parallel: 9
Parallel: 16
Parallel: 25
Parallel: 36
Parallel: 49
Parallel: 64
Parallel: 81
Press enter to finish

```

PLINQ został wyposażony w mechanizm opóźnionego wykonania zapytania (*Deferred Query Execution*), dokładnie tak samo, jak standardowy LINQ. Oznacza to, że konfiguracja zapytania nie pociąga za sobą konieczności jego wykonania po każdej zmianie, tylko w momencie, kiedy zostanie zgłoszone żądanie dostępu do danych.
Jeżeli istnieje potrzeba natychmiastowego wykonania zapytania, to wystarczy w danym miejscu dokonać konwersji zapytania PLINQ na dowolną standardową kolekcję. Do tego celu służą metody, takie jak : `ToList()`, `ToArray()`, `ToDictionary()`.

### Ograniczenia PLINQ

Niżej wymienione operatory zapobiegają zrównolegleniu zapytania PLINQ w sytuacji, gdy zmienia się indeks elementu w porównaniu do jego oryginalnej wartości w kolekcji źródłowej. 

* `Take`, `TakeWhile`, `Skip`, `SkipWhile`
* indeksowane wersje `Select`, `SelectMany` oraz `ElementAt`

Niektóre operatory mogą wykazywać się gorszą wydajnością niż ich podstawowe wersje LINQ:

* `Join`, `GroupBy`, `GroupJoin`, `Distinct`, `Union`, `Intersect` oraz `Except`
### Obsługa wyjątków w PLINQ

Każdy wyjątek (lub grupa wyjątków), który zostanie wygenerowany w zapytaniu PLINQ zostanie natychmiast opakowany do 
typu `System.AggregateException` jest zgłaszany na etapie wykonania zapytania, a nie jego definiowania.
Obiekt typu `AggregateException` jest używany do konsolidacji wielu wyjątków, które mogą zostać zgłoszone przez zadania TPL i PLINQ.
Obsługa zagregowanych wyjątków jest możliwa za pomocą metody `Handle()`:

```csharp
catch (AggregateException aggException)
{
    aggException.Handle(exception =>
        {
            Console.WriteLine("Przechwycony wyjątek typu:{0}", exception.GetType());

            //Zwrócenie False spowoduje propagację wyjątku
            return true;
        }
    );
}

```

### Anulowanie zapytań PLINQ

Wsparcie dla anulowania zapytań PLINQ oferuje typ `CancellationToken`. Odpowiednio skonfigurowany token należy przekazać do zapytania przy użyciu metody `WithCancellation()`.
Zdefiniowanie zadania wraz z tokenem i wykonanie zadania z obsługą błędu `OperationCanceledException`:

```csharp
const int N = 1000000;

var sourceData = Enumerable.Range(1, N).ToArray();
var tokenSource = new CancellationTokenSource();

var results = sourceData.AsParallel()
                .WithCancellation(tokenSource.Token)
                .Select(n => Math.Pow(n, 2));

```

    Task.Factory.StartNew(() =>
    {
        Thread.Sleep(500);
        tokenSource.Cancel();
    });
    try
    {
        results.ForAll(r =>
        {
            Console.WriteLine("Result: {0}", r);
        });
        Console.WriteLine("Koniec!");
    }
    catch (OperationCanceledException)
    {                
        Console.WriteLine("Anulowano wykonanie zadania!");
    }
### Agregacja w zapytaniach PLINQ

Agregacja jest procesem, w którym wiele elementów danych zostaje przetworzonych w pojedynczy wynik. Niektóre funkcje agregujące są tak często używane, że doczekały się dedykowanych procedur w LINQ/PLINQ, np. `Sum()`, `Average()`, `Count()` i inne.
Klasa ParallelEnumerable wspiera własną wersję funkcji `Aggregate()`, która jest zupełnie inną implementacją niż ta, którą oferuje LINQ.
Przykład:
1. Przygotowanie danych do przetwarzania:

```csharp
   var sourceData = Enumerable.Range(1, 10000).ToArray();

```

2. Wywołanie metody agregującej elementy tablicy:

```csharp
   double aggregateResult = sourceData.AsParallel().Aggregate(

```

3. Określenie pierwszego argumentu, którym jest fabryka wartości początkowych agregacji (*seedFactory*):

```csharp
   () => 0.0,

```

4. Określenie drugiego parametru, którym jest funkcja uruchamiana w ramach różnych zadań, iterujących po całej tablicy danych (*updateAccumulatorFunc*):

```csharp
   (subtotal, item) => subtotal += Math.Pow(item, 2),

```

5. Kolejny argument jest funkcją uruchamianą w momencie, kiedy wszystkie zadania ukończą przetwarzanie za pomocą pierwszej funkcji (*combineAccumulatorFunc*):

```csharp
   (total, subtotal) => total + subtotal,

```

6. Ostatnim argumentem jest procedura, która otrzymała wynik finalny a w której można wykonać ostateczne przekształcanie danych (*resultSelector*):

```csharp
   total => Math.Sqrt(total);

```

### Stopnie współbieżności

Domyślnie PLINQ wybiera optymalny stopień współbieżności wykonywanego zapytania. Aby wymusić inną wartość stopnia 
współbieżności należy po klauzuli `AsParallel()` wywołać `WithDegreeOfParallelism()` przekazując odpowiedni parametr.

```csharp
var result = sourceData.AsParallel().WithDegreeOfParallelism(8)...;

```

## Task Parallel Library

*Task Parallel Library* (TPL) bazuje na koncepcji współbieżnego wykonywania zadań. Termin task parallelism oznacza obsługę jednego lub kilku niezależnych zadań (*tasks*) uruchomionych jednocześnie, oddziałujących (najczęściej) na te same zasoby (*concurrency*). Zadanie określa asynchroniczną operację i powoduje utworzenie nowego wątku lub wykorzystanie wątku roboczego z puli wątków (`ThreadPool`) ale na wyższym poziomie abstrakcji. 
Stosowanie zadań dostarcza dwóch podstawowych korzyści:

* Bardziej efektywne i skalowalne wykorzystanie zasobów aplikacji i systemu. W tle zadania są rejestrowane w puli wątków, co gwarantuje maksymalne wykorzystanie dostępnych dla aplikacji wątków.
* Większa programowa kontrola niż w przypadku zwykłego wątku. Zadania wraz z frameworkiem tworzą bogate środowisko API, które wspiera anulowania zadań, wstrzymania i kontynuacje, rozbudowaną obsługę błędów, dokładne informacje o stanach zadań oraz wiele innych.

Ze względu na powyższe zalety w .NET 4.0 zadania (*tasks*) są preferowanym API do pisania wielowątkowego, asynchronicznego i wielozadaniowego kodu.
Zadania (*Tasks*) można uruchamiać na dwa sposoby:

* Niejawnie (`implicitly`) z wykorzystaniem klasy `Parallel`
* Jawnie (`explicilty`) z wykorzystaniem obiektów typu `Task`
### Klasa ``Parallel``

Klasa `Parallel` umożliwia wykonywanie zadań współbieżnie za pomocą trzech statycznych funkcji:

* `Parallel.Invoke()`
* `Parallel.For()`
* `Parallel.ForEach()`

Wszystkie powyższe metody blokują wątek, w którym zostały wykonane, do momentu zakończenia wykonania wszystkich zadań.
#### Parallel.Invoke()

Metoda `Parallel.Invoke()` obsługuje konwencjonalny mechanizm tworzenia i uruchamiania dowolnej liczby współbieżnych zadań. Najprostszym sposobem wykorzystania metody `Parallel.Invoke()`, jest przekazanie jej delegatu `Action`, dla każdego elementu, który ma zostać uruchomiony jako zadanie. W .NET 4.5 wykorzystuje się do tego najczęściej wyrażenia lambda, które zastępują konwencjonalny mechanizm delegatów. Największą zaletą takiego rozwiązania jest intuicyjne i szybkie generowanie kodu inline. 

```csharp
string[] contents = new string[2]; 

Parallel.Invoke(
    () => contents[0] = new WebClient().DownloadString("http://google.com"),
    () => contents[1] = new WebClient().DownloadString("http://bing.com")
);

```

Liczba zadań, które zostaną utworzone poprzez wywołanie metody `Parallel.Invoke()` niekoniecznie odpowiada liczbie przekazanych w parametrze delegatów, ponieważ TPL uruchamia wiele procedur optymalizacyjnych w czasie tej operacji.

#### ``Parallel.For`` oraz ``Parallel.ForEach``

`Parallel.For` oraz `Parallel.ForEach` są współbieżnymi odpowiednikami pętli `for` oraz `foreach`.
Aby równlogle wykonać pętlę `for`:

```csharp
for(int i = 0, i < 100; ++i)
    Call(i);

```

należy użyć:

```csharp
Parallel.For(0, 100, i => Call(i));

// lub 

Parallel.For(0, 100, Call);

```

Praktyczny przykład:

```csharp
var keyPairs = new string[6];

Parallel.For (0, keyPairs.Length,
              i => keyPairs[i] = RSA.Create().ToXmlString(true));

```

Pętla `foreach`:

```csharp
foreach(char c in "Hello World!")
    Call(c);

```

w wersji współbieżnej wygląda następująco:

```csharp
Parallel.ForEach("Hello Parallel World!", Call);

```

Aby wykorzystać informację o indeksie elementu w bieżącej iteracji należy wykorzystąć poniższą wersję 
pętli `ForEach()`:

```csharp
Parallel.ForEach ("Hello, world", (c, state, i) =>
{
   Console.WriteLine (c.ToString() + i);
});

```

Istnieje możliwość przerwania iteracji współbieżnej (analogicznie do ''break'' w iteracji klasycznej)
za pomocą obiektu typu `ParallelLoopState`. Zarówno `For` jak i `ForEach` maję przeciążone wersje, które akceptują parametr typu `Action<TSource, ParallelLoopState>`.

```csharp
Parallel.ForEach("Hello Parallel World!", (c, loopState) =>
{
    if (c == "P")
        loopState.Break();
    else
        Console.WriteLine(c);
});

```

Obiekt typu `ParallelLoopState` posiada dwie metody umożliwiające przerwanie pętli:

* `Break()` - gwarantuje ukończenie iteracji po wszystkich elementach sekwencji, które poprzedzają element iteracji bieżącej
* `Stop()` - wymusza koniec iteracji zaraz po zakończeniu iteracji bieżącej

##### Optymalizacja z wykorzystaniem tymczasowych zmiennych

W przypadku implementowania operacji agregacji za pomocą pętli `For` oraz `ForEach` możemy skorzystać z przeciążonej wersji, która optymalizuje wydajność korzystając ze zmiennych tymczasowych:

```csharp
object locker = new object();
double total = 0.0;

Parallel.For(1, 100000,
    () => 0.0,
    (i, loopState, localTotal) => localTotal + Math.Pow(i, 2),
    localTotal => { lock(locker) total += localTotal; }
);

```

## Jawne zarządzanie zadaniami - klasa ``Task``

Obiekt typu `Task` reprezentuje *operację asynchroniczną*. Umożliwia kontrolę nas trybem uruchomienia operacji oraz sprawdzenie statusu jej wykonania. Klasy umożliwiające jawne zarządzanie zadaniami umieszczone są w przestrzeni `System.Threading.Tasks`.

| Klasa                      | Cel                                          |
|----------------------------|----------------------------------------------|
| `Task`                     | Zarządza zadaniem                            |
| `Task<TResult>`            | Zarządza zadaniem, które zwraca wynik        |
| `TaskFactory`              | Umożliwia tworzenie zadań                    |
| `TaskFactory<TResult>`     | Umożliwia tworzenie zadań zwracających wynik |
| `TaskScheduler`            | Kontroluje szeregowanie zadań                |
| `TaskCompletionSource`     | Umożliwia kontrolę nad wykonywaniem zadań    |

### Tworzenie i uruchamianie zadań

* Statyczna metoda `Task.Run()` - wprowadzona w .NET Framework 4.5

```csharp
  Task.Run (() => Console.WriteLine ("Task"));

```

* Statyczna metoda `Task.Factory.StartNew`

```csharp
  Task.Factory.StartNew(() => Console.WriteLine ("Task"));

```

* Konstruktor `Task` 

```csharp
  new Task(() => 
  {
      Console.WriteLine("Task");
  }).Start();

```

#### Opcje tworzenia zadań

Tworząc obiekt zadania można określić preferowaną opcję sposobu wykonania zadania poprzez
jedną z wartości wyliczenia `TaskCreationOptions`:

* `None` - domyślna opcja heurestyki zadań
* `LongRunning` - sugeruje schedulerowi utworzenie osobnego wątku dla zadania (zalecane dla operacji I/O, które długo trwają)
* `PreferFairness` - zadania powinny być szeregowane w kolejności, w jakiej zostały uruchomione
* `AttachedToParent` - próbuje utworzyć zadanie podrzędne względem bieżącego zadania (zadania w ramach zadań)

Wartości `TaskCreationOptions` mogą być łączone bitowo, tworząc kombinację różnych opcji.

```csharp
Task parent = Task.Run(() =>
{
    Console.WriteLine("Starting Parent");

    Task.Factory.StartNew(() => Console.WriteLine("Detached"));

    Task.Factory.StartNew(() => 
    {
        Console.WriteLine("Child");
    }, TaskCreationOptions.AttachedToParent);
};

```

#### Czekanie na zakończenie zadania

Aby zaczekać na wykonanie zadania reprezentowanego przez obiekt `Task` należy wywołać blokującą 
metodę `Wait()`:

```csharp
Task task = Task.Run (() =>
{
    Thread.Sleep (2000);
    Console.WriteLine ("Finished");
});

Console.WriteLine (task.IsCompleted);  // False

task.Wait();  // Blocks until task is complete

```

Jeśli zadanie ma zadania podrzędne (child tasks) `Wait()` czeka na zakończenie wszystkich zadań podrzędnych.
Istnieje możliwość wygodnego oczekiwania na zakończenie kilku zadań przy pomocy metod:

* `WaitAll()` - blokuje do czasu zakończenie wszystkich zadań

```csharp
  var tasks = new Task[3] {
      Task.Factory.StartNew(()=> new WebClient().DownloadString("http://google.com")),
      Task.Factory.StartNew(()=> new WebClient().DownloadString("http://bing.com")),
      Task.Factory.StartNew(()=> new WebClient().DownloadString("http://yahoo.com")) };

  Task.WaitAll(tasks);

```

* `WaitAny()` - blokuje do zakończenia jednego z podanych zadań
### Zadania zwracające wynik

Klasa `Task<TResult>` umożliwia tworzenie asynchronicznych zadań, które w zwracają wartość.

```csharp
Task<int> task = Task<int>.Run(() => { Console.WriteLine("Start"); return 42; });

// ...

int result = task.Result; // blokuje do momentu obliczenia zwracanej wartości

```

`Task<TResult>` reprezentuje przyszły wynik obliczenia wartości przez funkcję asynchroniczną.
Właściwość `Result` zwraca obliczoną wartość.
### Obsługa wyjątków dla zadań

Zadania propagują wyjątki zgłaszane przez wykonywane za pośrednictwem delegata funkcje.
Jeśli zadanie rzuci wyjątek, a kod klienta wywoła `Wait()` lub `Result` przechwycony przez zadanie wyjątek zostanie automatycznie przerzucony do kodu klienta w postaci wyjątku typu `AggregateException`.
Stan zadania może być testowany za pomocą właściwości `IsFaulted` lub `IsCanceled`. 

* `IsCanceled` zwraca `true` jeśli wykonanie zadania zostało przerwane - rzucony został wyjątek `OperationCanceledException`
* `IsFaulted` zwraca `true`, jeśli rzucony został inny typ wyjątku. Wyjątek może zostać odczytany poprzez właściwość `Exception`

Nieobsłużone wyjątki zadań dla których nie wywołano metod `Wait()` lub `Result` (tzw. *set-and-forget*) są nazywane
"niezaobserwowanymi wyjątkami". 

* W CLR 4.0 powodowały one ostatecznie przerwanie wykonywania programu - CLR rzucało ponownie wyjątek w trakcie działania finalizatora wątku. 
* CLR 4.5 nie rzuca ponownie wyjątku - jest on zignorowany

Można globalnie zarejestrować obsługę "niezaobserwowanych wyjątków" poprzez statyczne zdarzenie `TaskScheduler.UnobservedException`.
### Anulowanie zadań

Klasa `Task` wspiera proces kooperatywnego anulowania zadań, który jest w pełni obsługiwany przez 
typ `System.Threading.CancellationTokenSource`. Ten ostatni generuje właściwość `Token`, która służy jako parametr konstruktora klasy `Task`.
Ogólny wzorzec implementacji kooperatywnego anulowania zdań:

* Należy utworzyć instancję `CancellationTokenSource` - token
* Przekazać token do wszystkich zadań lub wątków, które mogą być anulowane
* Wywołać metodę `IsCancellationRequested()` z operacji, której został przekazany token. Zapewnić dla każdego 

  zadania mechanizm odpowiedzi na żądanie anulowania operacji. Sposób anulowania zależy od implementacji klienta.

* Wywołać `Cancel`, aby zapewnić rozgłoszenie anulowania zadania (to ustawia flagę `IsCancellationRequested` we wszystkich zadaniach na `true`)
* Wywołać `Dispose()`, kiedy token nie jest już potrzebny.

```csharp
var tokenSource = new CancellationTokenSource();

var tasks = new Task[] {
    Task.Factory.StartNew(() => 
    {
        for (var i=0;i<10;i++) 
        {
            Console.WriteLine("Zadanie 1:" + i);
            if (tokenSource.IsCancellationRequested) 
                throw new OperationCanceledException("Wykryto anulowanie w zadaniu 1");
            System.Threading.Thread.Sleep(200);
        };
        Console.WriteLine("Zadanie 1 anuluje token");
        tokenSource.Cancel();
    }, tokenSource.Token),

    Task.Factory.StartNew(() => 
    {
        for (var i=0;i<10;i++) 
        {
            Console.WriteLine("Zadanie 2:" + i);
            if (tokenSource.IsCancellationRequested) 
                throw new OperationCanceledException("Wykryto anulowanie w zadaniu 2");
            System.Threading.Thread.Sleep(200);
        };
        Console.WriteLine("Zadanie 2 anuluje token");
        tokenSource.Cancel();
   }, tokenSource.Token)
};

try
{
    Task.WaitAny(tasks);
}
catch (AggregateException) {}

```

Anulowanie tokenu nie powoduje automatycznego zakończenia zadań powiązanych z tym tokenem.  Każde zadanie musi w odpowiedni sposób odczytywać właściwość `CancellationToken.IsCancellationRequested` i właściwie na nią reagować.
### Mechanizm kontynuowania zadań

Mechanizm kontynuowania zadań (*Task Continuations*) – przydatna implementacja, która pozwala na sekwencyjne wykonywanie zadań jedno po drugim. Poprzedzające zadanie musi być ukończone zanim kolejne będzie mogło być rozpoczęte. Zaletą takiego rozwiązania jest również to, że wynik poprzedniego zadania może być parametrem wejściowym dla kolejnego zadania.
Przykładowe metody wykonywane sekwencyjnie:

```csharp
static byte[] GetFileData()
{
     return Encoding.ASCII.GetBytes("Przykładowy tekst");
}

static int[] Analize(byte[] input)
{
     return input.Select(p => (int)p).ToArray();
}

static string Summarize(int[] input)
{
     return Encoding.ASCII.GetString(input.Select(p => (byte)p).ToArray());
}

```

Wynik poprzedzającej metody będzie zarazem parametrem wejściowym kolejnej, przy założeniu, że programista ma pełną kontrolę na kolejnością wykonywania zadań.
Pierwszy sposób wykonywania zadań sekwencyjnych:

```csharp
var getData = new Task<byte[]>(() => GetFileData());
var analyzeData = getData.ContinueWith(x => Analize(x.Result));
var reportData1 = analyzeData.ContinueWith(y => Summarize(y.Result));

getData.Start();

var result1 = reportData1.Result;

```

Drugi sposób wykonywania zadań sekwencyjnych:

```csharp
var reportData2 = 
    Task.Factory.StartNew(() => GetFileData()).
         ContinueWith((x) => Analize(x.Result)).
            ContinueWith((y) => Summarize(y.Result));

var result2 = reportData2.Result;

```

Druga metoda jest bardziej logiczna i intuicyjna dzięki zastosowaniu lambdy. Obie procedury mają taki sam narzut czasowy i mogą być stosowane wymiennie.
Oprócz prostej sekwencyjności TPL dostarcza jeszcze rozwiązania bardziej rozbudowanego, które pozwala na uruchamianie kolejnych zadań pod pewnymi warunkami np. gdy wszystkie określone zadania z puli zadań zostaną ukończone (`ContinueWhenAll`) lub gdy którekolwiek zadanie z puli zadań zostanie ukończone (`ContinueWhenAny`). Metody te są dostępne z poziomu klasy `TaskFactory`.
#### Opcje kontynuacji zadań

Domyślnie, zadania kontynuujące mogą zostać uruchomione w innych wątkach. Opcja `TaskContinuationOptions.ExecuteSynchronously` umożliwia użycie tego samego wątku co poprzednik.
Kontynuacje domyślnie są uruchamiane bezwarunkowo (zarówno w przypadku pomyślnego zakończenia zadania lub rzucenia 
wyjątku). Zachowanie to można zmienić wykorzystując zbiór flag zdefiniowanych w wyliczeniu `TaskContinuationOptions`.
    - `NotOnRanToCompletion`
    - `NotOnFaulted`
    - `NotOnCanceled`
    - `OnlyOnRanToCompletion  = NotOnFaulted | NotOnCanceled`
    - `OnlyOnFaulted = NotOnRanToCompletion | NotOnCanceled`,
    - `OnlyOnCanceled = NotOnRanToCompletion | NotOnFaulted`

* kontynuacja w przypadku zgłoszenia wyjątku przez zadanie poprzednika

```csharp
  Task task1 = Task.Factory.StartNew(() => { throw null; });

  Task error = task1.ContinueWith(
                   ant => Console.Write (ant.Exception),
                   TaskContinuationOptions.OnlyOnFaulted);

```

* kontunuacja w przypadku prawidłowego zakończenia zadania poprzednika

```csharp
 Task ok = task1.ContinueWith(
               ant => Console.Write ("Success!"),
               TaskContinuationOptions.NotOnFaulted);                       

```

#### Kontynuacja dla wielu poprzedników

* Istnieje możliwość ustawienia kontynuacji dla wielu uruchomionych zadań:

```csharp
var task1 = Task.Run (() => Console.Write ("X"));
var task2 = Task.Run (() => Console.Write ("Y"));

var continuation = Task.Factory.ContinueWhenAll (
                        new[] { task1, task2 }, tasks => Console.WriteLine ("Done"));

```

* oraz wielu zadań kontynuujących dla jednego poprzednika

```language
  var t = Task.Factory.StartNew (() => Thread.Sleep (1000));
  t.ContinueWith (ant => Console.Write ("X"));
  t.ContinueWith (ant => Console.Write ("Y"));                     

```

#### Kontynuacja dla UI

.NET Framework posada dwie implemenntacje schedulerów:

* default scheduler - pracuje z pulą wątków CLR
* synchronization context scheduler - zaprojektowany do pracy z wątkami w WPF/Windows Forms

Model wątków w WPF/Windows Forms wymaga, aby elementy UI (kontrolki) były uaktualniane tylko z wątku, w którym zostały utworzone. *Synchronization context scheduler* łatwo umożliwia rozwiązanie tego problemu.

```csharp
_uiScheduler = TaskScheduler.FromCurrentSynchronizationContext();

Task.Run(() => BackgroundTask())
    .ContinueWith(bt => lblResult.Content = bt.Result, _uiScheduler);

```

## Metody asynchroniczne - ``async`` i ``await``

Słowo kluczowe `await` upraszcza kontynuowanie zadań wykonywanych asynchronicznie.
Kod: 

```csharp
var result = await expression;
statements(result);

```

kompilator zamienia w:

```csharp
var awaiter = expression.GetAwaiter();
awaiter.OnCompleted (() =>
{
    var result = awaiter.GetResult();
    statement(s);
});

```

Przykład użycia funkcji `async`:

```csharp
Task<int> GetPrimesCountAsync (int start, int count)
{
    return Task.Run(() =>
        ParallelEnumerable.Range(start, count).Count (n =>
            Enumerable.Range(2, (int)Math.Sqrt(n)-1).All (i => n % i > 0)));
}

async void DisplayPrimesCount()
{
    int result = await GetPrimesCountAsync(2, 100000);
    Console.WriteLine(result);
}

```

Metody poprzedzone słowem kluczowym `async` są nazywane funkcjami asynchronicznymi. Modyfikator `async` może być stosowany tylko dla metod (lub funkcji lambda), które zwracają `void`, `Task` lub `Task<TResult>`.
Po napotkaniu słowa kluczowego `await` następuje wyjście z funkcji do wywołującego. Przed opuszczeniem funkcji CLR podłącza kontynuację zadania, dając tym samym gwarancję, że gdy zadanie zostanie ukończone, wykonanie kodu wróci do miejsca wyjścia. W przypadku, gdy zadanie, na które czekamy zakończy się wyjątkiem, zostanie on odrzucony do wywołującego. 
### Ograniczenia ``await``

Wyrażenie `await` może wystąpić niemal w każdym miejscu kodu z wyjątkiem:

* wyrażenia `lock`
* kontekstu `unsafe`
* metodzie, od której zaczyna się wykonanie zestawu (`Main()`)

```csharp
async void DisplayPrimeCounts()
{
    for (int i = 0; i < 10; i++)
        Console.WriteLine (await GetPrimesCountAsync (i*1000000+2, 1000000));
}

```

Jeśli funkcja asynchroniczna jest wykonywana w wątku UI, kontekst synchronizacji zapewnia, że wykonanie po zakończeniu zadania zostanie wznowione w tym samym wątku (wątku UI).
Przykład funkcji asynchronicznej jako handlera zdarzenia UI:

```csharp
async void Go()
{
    _button.IsEnabled = false;
     for (int i = 1; i < 5; i++)
         _results.Text += await GetPrimesCountAsync (i * 1000000, 1000000) +
                              " primes between " + (i*1000000) + " and " + ((i+1)*1000000-1) +
                              Environment.NewLine;
    _button.IsEnabled = true;
}

class TestUI : Window
{
    Button _button = new Button { Content = "Go" };
    TextBlock _results = new TextBlock();

    public TestUI()
    {
       var panel = new StackPanel();
       panel.Children.Add (_button);
       panel.Children.Add (_results);
       Content = panel;
       _button.Click += (sender, args) => Go();
    }

```

### Metody async zwracające void

* Metody `async void` działają jak metody *fire-and-forget*

  - wywołujący nie jest w stanie dowiedzieć się kiedy zakończyła się operacja synchroniczna
  - wywołujący nie jest w stanie złapać wyjątku rzuconego z takiej funkcji (taki wyjątek zostanie zgłoszony ponownie
    w pętli zdarzeń UI

* Zalecenia odnośnie stosowania `async void`

  - należy stosować je tylko jako handlery zdarzeń
  - w pozostałych przypadkach powinno się stosować metody async zwracające `Task`
  - async lambda może działać i powodować podobne problemy co `async void`
## Kolekcje współbieżne

Kolekcje są najczęstszym sposobem na współdzielenie danych w ramach różnych zadań w programie. Często istnieje potrzeba współbieżnego przetwarzania zawartości kolekcji lub gromadzenia wyników dostarczanych przez różne współbieżne zadania.

.NET 4.5 oferuje specjalne kolekcje, które umożliwiają osiągnięcie tego celu. Definicja tych kolekcji znajduje się w przestrzeni nazw `System.Collections.Concurrent`. Obiekty tych kolekcji są `thread-safe`.

| Rodzaj problemu                                                            | Kolekcja rozwiązująca problem               |
|----------------------------------------------------------------------------|---------------------------------------------|
| Bezpieczna kolekcja danych realizująca strategię FIFO (First-in, First-out) | System.Threading.Tasks.ConcurrentQueue      |
| Bezpieczna kolekcja danych realizująca strategię First-in, Last-out       | System.Threading.Tasks.ConcurrentStack      |
| Bezpieczna kolekcja danych bez specyfikacji kolejności                    | System.Threading.Tasks.ConcurrentBag        |
| Bezpieczna słownikowa kolekcja danych                                      | System.Threading.Tasks.ConcurrentDictionary |

### Concurrent Queue

`ConcurrentQueue` realizuje strategię FIFO, co oznacza, że kiedy elementy zostaną ściągnięte z kolejki, to zostanie odtworzona kolejność w jakiej zostały do kolejki dodane.

Podstawowe metody do obsługi `ConcurrentQueue`:

| Metoda            | Opis                                                                                                       |
|-------------------|------------------------------------------------------------------------------------------------------------|
| Enqueue(T)        | Dodaje element do kolekcji.                                                                                |
| TryPeek(out T)    | Próbuje pobrać pierwszy element kolekcji bez jego usuwania. Zwraca `true`, jeżeli operacja się powiodła.  |
| TryDequeue(out T) | Próbuje pobrać i usunąć pierwszy element z kolekcji. Zwraca `true`, jeżeli operacja powiodła się.         |

```csharp
// Krok 1. Utworzenie i zainicjalizowanie kolekcji.
var sharedQueue = new ConcurrentQueue<int>();
for (var i = 0; i < 1000; i++)
{
    sharedQueue.Enqueue(i);
}

// Krok 2. Współbieżne usuwanie elementów kolekcji.
var itemCount = 0;
var tasks = new Task[10];
for (var i = 0; i < tasks.Length; i++)
{
    tasks[i] = new Task(() =>
    {
        int queueElement;
        if (sharedQueue.TryDequeue(out queueElement))
        {
            Interlocked.Increment(ref itemCount);
        }
    });
    tasks[i].Start();
}

```

### ConcurrentStack

`ConcurrentStack` – realizuje strategię LIFO (last in, first out), która polega na tym, że ściągany jest zawsze element, który został do kolekcji dodany jako ostatni. 

| Metoda                                       | Opis                                                            |
|----------------------------------------------|-----------------------------------------------------------------|
| Push(T)                                      | Dodaje element do stosu                                         |
| PushRange(T[]), PushRange(T[], int, int)    | Dodaje tablicę elementów do stosu                               |
| TryPeek(out T)                               | Próbuje zwrócić pierwszy element stosu bez jego usuwania        |
| TryPop(out T)                                | Próbuje zwrócić pierwszy element stosu jednocześnie usuwając go |
| TryPopRange(out T[]), TryPopRange(out T[], int, int) | Jw. Tylko wiele elementów jednocześnie                          |

```csharp
// Krok 1. Utworzenie i zainicjalizowanie kolekcji.
var sharedStack = new ConcurrentStack<int>();
for (var i = 0; i < 1000; i++)
{
    sharedStack.Push(i);
}

// Krok 2. Współbieżne usuwanie elementów kolekcji.
var tasks = new Task[10];
for (var i = 0; i < tasks.Length; i++)
{
    tasks[i] = new Task(() => {
        int queueElement;
        if (sharedStack.TryPop(out queueElement))
        {
            Interlocked.Increment(ref itemCount);
        }
    });
    tasks[i].Start();
}

```

### ConcurrentBag

`ConcurrentBag` jest klasą implementującą nieuporządkowaną listę elementów, gdzie nie ma gwarancji co do kolejności zwróconych elementów. 

| Metoda         | Opis                                               |
|----------------|----------------------------------------------------|
| Add(T)         | Dodaje element do kolekcji                         |
| TryPeek(out T) | Próbuje zwrócić element z kolekcji bez usuwania go |
| TryTake(out T) | Próbuje zwrócić i usunąć element z kolekcji.       |

```csharp
// Krok 1. Utworzenie i zainicjalizowanie kolekcji.
var sharedBag = new ConcurrentBag<int>();
for (var i = 0; i < 1000; i++)
{
    sharedBag.Add(i);
}

// Krok 2. Współbieżne usuwanie elementów kolekcji.
var tasks = new Task[10];
for (var i = 0; i < tasks.Length; i++)
{
    tasks[i] = new Task(() =>{
        int queueElement;
        if (sharedBag.TryTake(out queueElement))
        {
            Interlocked.Increment(ref itemCount);
        }
    });
    tasks[i].Start();
}    

```

### ConcurrentDictionary

`ConcurrentDictionary` jest klasą implementującą kolekcję słownikową (para klucz-wartość).

| Metoda                                  | Opis                                                                                                                                                                 |
|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TryAdd(TKey, TVal)                      | Próbuje dodać nową parę klucz-wartość do kolekcji                                                                                                                    |
| TryGetValue(TKey, out TVal)             | Próbuje odczytać wartość skojarzoną z danym kluczem                                                                                                                  |
| TryRemove(TKey, out TVal)               | Próbuje usunąć parę klucz-wartość z kolekcji                                                                                                                         |
| TryUpdate(TKey, TVal, TVal)             | Próbuje zmienić wartość skojarzoną z danym kluczem                                                                                                                   |
| ContainsKey(TKey)                       | Zwraca `true`, jeżeli w kolekcji znajduje się para klucz-wartość skojarzona z danym kluczem                                                                          |
| AddOrUpdate(TKey, TValue, Func<TKey, TValue, TValue>) | Dodaje parę klucz/wartość do słownika jeśli słownik nie przechowuje jeszcze klucza, jeśli klucz istnieje wartość jest uaktualniona przy pomocy podanej funkcji      |
| GetOrAdd(TKey, Func<TKey, TValue>)      | Dodaje parę/klucz wartość do słownika jeśli słownik nie przechowuje jeszcze klucza przy pomocy podanej funkcji                                                       |

Poniższy scenariusz nie gwarantuje pobrania oczekiwanych danych ze słownika:
1.  Sprawdzenie, czy wpis istnieje za pomocą metody `ContainsKey()`
2.  Pobranie wpisu w kolejnej linii kodu za pomocą metody `TryGetValue()`
Między wywołaniem jednej i drugiej metody inne zadanie mogło zmodyfikować lub usunąć wpis.  

```csharp
// Krok 1. Zdefiniowanie klasy pomocniczej.
class BankAccount
{
    public int Balance { get; set; }
}

// Krok 2. Zdefiniowanie odpowiednich zadań.
var account = new BankAccount();
var sharedDict = new ConcurrentDictionary<object, int>();
var tasks = new Task<int>[10];

for (var i = 0; i < tasks.Length; i++) 
{
    sharedDict.TryAdd(i, account.Balance);

    tasks[i] = new Task<int>((keyObj) => {
        int currentValue;
        for (var j = 0; j < 1000; i++) 
        {
            if (sharedDict.TryGetValue(keyObj, out currentValue))  
            {
                sharedDict.TryUpdate(keyObj, currentValue + 1, currentValue);
            }                            
        }

        if (!sharedDict.TryGetValue(keyObj, out currentValue)) 
        {
            throw new Exception("Wpis {0} nie znaleziony "+ keyObj);
        }
        return currentValue;
    }, i);
    tasks[i].Start();
}

// Krok 3. Pobranie wyników zwróconych przez poszczególne zadania.
for (var i = 0; i < tasks.Length; i++)
{
    account.Balance += tasks[i].Result;
}
Console.WriteLine("Wartość oczekiwana {0} wartość uzyskana {1}", 10000,
                  account.Balance);
```

Powyższy algorytm jest przykładem realizacji wzorca izolacji danych, który pozwala na wielokrotny i jednoczesny dostęp zadań do tego samego zasobu, przy czym izoluje stan zasobu w ramach każdego zadania.
