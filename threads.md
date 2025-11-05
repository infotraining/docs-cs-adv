# Programowanie współbieżne
## Wątki

**Wątki** – wydzielone jednostki zadaniowe tworzone przez proces macierzysty w celu wykonywania pewnych operacji. Każdy zestaw .exe rozpoczyna działanie tworząc wątek główny, który służy jako punkt wejścia do aplikacji (metoda Main()). Wątek główny może tworzyć nowe (zarządzane) wątki przy pomocy klasy Thread.
Fizycznie wątek składa się z:

* rejestrów procesora
* stosu wywołań
* lokalnej pamięci wątku (**Thread Local Storage** – TLS)

Wątki i domeny aplikacji:
![image](_images/app-domain-threads.*)

Wielowątkowość jest realizowana przez system operacyjny dzięki algorytmowi szeregowania (thread scheduler), który ciągle przełącza działające wątki, zezwalając na ich działanie przez określony, skończony przedział czasu (time-slice). 
Algorytm szeregowania zapewnia poprawną alokację czasową wątków. Wątki oczekujące lub zablokowane nie konsumują czasu CPU. Kolejność uruchamiania wątków oparta jest na priorytetach. Wątek jest wywłaszczony jeśli wyczerpie się jego przedział czasu lub wątek o wyższym priorytecie rozpocznie swoje wykonywanie. Jeśli w tym czasie przydzielonym przez system operacyjny wątek nie zakończy swojego działania, trzeba zapisać informację o jego stanie (rejestry, wskaźniki stosu itp.). Informacje o stanie wątku przechowywane są w TLS.
### Stan wątku

Każdy wątek znajduje się w jednoznacznym stanie określanym przez właściwość ThreadState obiektu reprezentującego wątek:

* **Unstarted** – obiekt `Thread` został przydzielony, ale nie rozpoczął jeszcze działania
* **Running** – metoda `ThreadStart` danego wątku jest albo gotowa do wykonania albo właśnie wykonywana
* **Aborted** – wątek został przerwany albo indywidualnie, albo jako część procesu zwalniania domeny aplikacji
* **Stopped**  - wątek natywny zakończył się i nie jest już wykonywany
* **Suspended** – wykonywanie wątku zostało zawieszone
* **WaitSleepJoin** – wykonywanie wątku jest obecnie zablokowane w kodzie zarządzanym i oczekuje na wystąpienie pewnych warunków. Jest to inicjowane automatycznie, gdy wątek korzysta z metod `Monitor`.`Enter`, `Thread.Sleep`, `WaitHandle.WaitOne`

![image](_images/thread-state.*)

Stan wątku może zawierać także wartości informacyjne:

* `AbortRequested` – żądanie przerwania wątku zostało wysłane, ale wątek jeszcze nie odpowiedział poprzez rzucenie wyjątku `ThreadAbortException`
* `StopRequested` – żądanie zatrzymania wątku zostało wysłane przez podsystem wątków CLR
* `SuspendRequested` – żądanie zawieszenia wykonywania wątku zostało wysłane
## Klasa ``Thread``

Wątki są tworzone za pomocą konstruktora klasy `Thread`. Do konstruktora przekazany klasy zostaje delegat  `ThreadStart` do metody, która ma zostać uruchomiona w osobnym wątku. 

```csharp
public delegate void ThreadStart();

```

Uruchomienie wątku odbywa się poprzez wywołanie metody `Start()`. 
Wątek wykonuje swoją pracę dopóki jego metoda nie zakończy działania instrukcją return. Właściwość `IsAlive`  zwraca true, jeśli wykonywanie wątku nie zakończyło się, czyli jeśli stan wątku jest różny od `Stopped` lub `Aborted`. Wątek, który zakończył swoją pracę, nie może być ponownie uruchomiony.
Programowe tworzenie wątku:

```csharp
static void WriteY()
{
    while (true)
        Console.Write("Y");
}

static void Main(string[] args)
{
    // utworzenie nowego wątku
    // parametr: delegat do metody void X()
    Thread thd = new Thread(WriteY);

    thd.Start(); // uruchomienie wątku
    while (true)
        Console.Write("X");
}

```

### Wykonywanie wątków

Metoda `Thread.Start` rozpoczyna wykonywanie wątku.
Wykonywanie jest kontynuowane, dopóki nie nastąpi jedno z kilku zdarzeń:

* Wyszczególniona metoda rozpoczynająca wątek zakończy się – stanem wątku będzie `Stopped`.
* Zgłoszono żądanie przerwania wątku – albo jawnie, albo w ramach zwalniania domeny `AppDomain`.   

  Powoduje to przejście w stan `Aborted`.

* Wywołano operację blokującego oczekiwania (np. `Monitor.Wait`, 

  sprzeczny `Monitor.Enter`, `WaitHandle.WaitOne`), `Thread.Sleep` lub `Thread.Join`. Stan wątku ustawiony jest na `WaitSleepJoin`, wykonywanie jest zablokowane i rozpoczyna się na nowo, kiedy żądany warunek jest spełniony.

* Wykonywanie wątku jest zawieszone - wywołanie `Suspend` i zmiana stanu na `Suspended`.
### Nazywanie wątku

Wątek może zostać nazwany. Właściwość `Name` ułatwia identyfikację wątku.

```csharp
static void Go() 
{
    Console.WriteLine ("Hello from " + Thread.CurrentThread.Name);
}

public static void Main() 
{
    Thread.CurrentThread.Name = "main";
    Thread worker = new Thread (Go);
    worker.Name = "worker";
    worker.Start();
    Go();
}

```

### Wątki pierwszoplanowe oraz wątki tła

**Wątki pierwszoplanowe** (*foreground threads*) – utrzymują aplikację przy życiu tak długo, dopóki aktywny jest choć jeden z nich.
**Wątki tła** (*background threads*) – nie utrzymują aplikacji przy życiu. Są natychmiastowo przerywane gdy wszystkie wątki pierwszoplanowe zakończą swoją pracę.

```csharp
Thread thd = new Thread(() => DoSomeWork());
thd.IsBackground = true;
thd.Start();

```

Zmiana wątku z pierwszoplanowego na wątek tła nie zmienia jego priorytetu oraz statusu w algorytmie szeregowania (CPU scheduler). Częstym błędem jest obecność zapomnianych wątków pierwszoplanowych, co powoduje, że aplikacja nie może zostać zamknięta.
### Priorytety wątku

Kolejność uruchamiania wątków oparta jest na priorytetach. Jeśli wątek jest uruchomiony, a powstaje wątek o wyższym priorytecie, działający wątek zostaje wstrzymany, aby umożliwić działanie nowemu wątkowi. Priorytet określa też, jak duży jest przydział czasu dla danego wątku w porównaniu do pozostałych aktywnych wątków.
Kontrolująca priorytet właściwość `Priority` może przyjmować jedną z wartości wyliczenia:

```csharp
enum ThreadPriority { Lowest, BelowNormal, Normal, AboveNormal, Highest }

```

Nawet wątki o najwyższym priorytecie mogą zostać zablokowane przez inne wątki. Podnoszenie priorytetów wątków może spowodować ich rywalizację z wątkami systemowymi i pogorszyć ogólną wydajność systemu. System kontroluje, kiedy działa każdy wątek. Jeśli wątek nie jest uruchamiany przez pewien czas, system zwiększa jego priorytet, aby umożliwić jego wykonanie.
### Obsługa sytuacji wyjątkowych

Bloki `try/catch/finally` w bloku tworzącym wątek nie mają znaczenia dla kodu wątku.

```csharp
public static void Main() 
{****
   try 
   {
      new Thread (Go).Start();
   }
   catch (Exception ex) 
   {
      // catch nigdy nie zostanie wywołany
      Console.WriteLine ("Exception!");
   }
}

static void Go() { throw null; }

```

### Usypianie wątku

Wywołanie `Thread.Sleep()` blokuje wątek przez podany  przedział czasu (jako parametr) lub do przerwania wątku.

```csharp
static void Main() 
{
    Thread.Sleep(0); // wymusza wywłaszczenie wątku 
    Thread.Sleep(1000); // wstrzymuje wykonanie na 1000 ms
    Thread.Sleep(TimeSpan.FromHours (1)); // wstrzymanie na 1h
    Thread.Sleep(Timeout.Infinite); // wstrzymanie dopóki wątek nie zostanie
                                     // przerwany
}

```

`Thread.Sleep(0)` pozwala na przełączenie się na wątek będący następnym w kolejce szeregowania.
`Thread.Yield()` działa podobnie, z tym, że przełączenie na inny wątek odbywa się w ramach tego samego procesora.
### Czekanie na zakończenie wątku

Aby zaczekac na zakończenie innego wątku należy wywołac nim metodę `Join()`.
Wątek, w którym wywołana jest metoda`Join` jest blokowany.

```csharp
class JoinDemo 
{
    static void Main() 
    {
        Thread thd = new Thread(() => { Console.ReadLine(); });
        thd.Start();

        thd.Join(); // oczekuje na zakończnie wątku thd
        Console.WriteLine ("Thread t's ReadLine complete!");
    }
}

```

### Przerywanie wątku

Wątek może zostać przedwcześnie przerwany na dwa sposoby:

* `Thread.Interrupt()`- pozwala na asynchroniczne przerwanie wątku zablokowanego w stanie `WaitSleepJoin`. Takie przerwanie powoduje wyjątek ThreadInterruptedException
* `Thread.Abort()` – przerwanie (zatrzymanie) wątku w bezpieczny sposób. Operacja przerwania wątku wstawia `ThreadAbortedException` do bieżącej linii wykonywania. Wyjątki przerwania są nieodwołalne.

Przerwanie wątku dokonane jest z poziomu innego wątku – wątek będący w stanie oczekiwania nie jest w stanie wykonać jakiejkolwiek operacji zmieniającej jego stan
## Współdzielenie danych

CLR przypisuje każdemu wątkowi własną przestrzeń stosu. Zmienne lokalne wątku są izolowane. 
Wątki współdzielą dane, jeżeli mają (dzielą) wspólne referencje do tej samej instancji obiektu. Statyczne pola w klasach są częstym mechanizmem współdzielenia danych pomiędzy wątki.
Współdzielenie danych, które mogą zmieniac swój stan (*mutable*), przez wiele wątków jednocześnie stwarza problemy. Do najczęściej występujących problemów należą:

* Wyścigi (*race conditions*)
* Zakleszczenia (*deadlocks*)

Stosowanie sekcji krytycznych umożliwia ochronę przed niektórymi błędami współbieżności (np. *race condition*). Sekcja krytyczna umożliwia chroniony dostęp do bloków kodu w taki sposób, że tylko jeden wątek może znajdować się takiej sekcji w danym czasie. 
Powszechną implementacją sekcji krytycznych jest użycie blokad. Jeśli kod chce zmodyfikować lub odczytac współdzielony obszar, zostaje mu przydzielona blokada. Wprowadzenie blokady jest celowym zakazem jednoczesnego wykonywania wątków w tej samej sekcji.
### Synchronizacja i blokady

* Proste sposoby blokowania:
  * `Sleep()` – blokuje wątek przez określony przedział czasu
  * `Join` – czeka, aż inny watek zakończy swoje działanie
* Obiekty blokad umożliwiające synchronizację:

| Sposób             | Cel                                                                                         | Cross-Process? | Efektywność |
|--------------------|---------------------------------------------------------------------------------------------|----------------|-------------|
| `Monitor` / `lock` | Sekcja krytyczna – tylko jeden wątek ma dostęp do zasobu lub sekcji kodu                    | Nie            | Szybki      |
| `Mutex`            | Mutual exclusion – zapobiega dostępowi do współdzielonych zasobów wielu wątków jednocześnie | Tak            | Średni      |
| `Semaphore`        | Ogranicza ilość wątków, które mogą uzyskać dostęp do tego samego współdzielonego zasobu     | Tak            | Średni      |

* Konstrukcje sygnalizujące:

| Sposób              | Cel                                                                            | Cross-Process? | Efektywność |
|---------------------|--------------------------------------------------------------------------------|----------------|-------------|
| `EventWaitHandle`   | Pozwala wątkowi na czekanie dopóki nie otrzyma on sygnału od innego wątku      | Tak            | Średni      |
| `Wait` oraz `Pulse` | Pozwala wątkowi czekać na spełnienie warunków zdefiniowanych przez użytkownika | Nie            | Średni      |

#### Blokowanie

Kiedy wątek czeka lub pauzuje w wyniku użycia jednej z wyżej wymienionych konstrukcji, to znajduje się w stanie blokady. Zablokowany wątek natychmiastowo zwalnia zasoby CPU i przechodzi w stan `ThreadState.WaitSleepJoin`. Informuje o tym właściwość `ThreadState` wątku.

```csharp
bool blocked = (someThread.ThreadState & ThreadState.WaitSleepJoin) != 0;

```

Odblokowanie wątku jest możliwe na cztery sposoby:

* Gdy warunek blokowania zostanie spełniony
* Upłynie czas przypisany blokadzie
* Wątek zostanie przerwany przez `Thread.Interrupt()`
* Wątek zostanie przerwany (anulowany) przez `Thread.Abort()`
### Monitory oraz bloki blokad

Klasa `Monitor` umożliwia zablokowanie obiektu przez wątek. Metody tej klasy służą do kontroli dostępu wątków do całego obiektu lub wybranych fragmentów jego kodu.
Najczęściej używane metody:

* `Monitor.Enter(object obj)` – próbuje wkroczyć do monitora określonego obiektu i blokuje się, dopóki próba nie zakończy się sukcesem
* `Monitor.Exit(object obj)` – powoduje opuszczenie monitora obiektu oraz zgłoszenie `SynchronizationLockException`, jeśli wątek wywołujący nie znajduje się w monitorze obiektu `obj`

```csharp
static int nextId;
static object locker = new object();

static int NextId() 
{
    try 
    {
        Monitor.Enter(locker);
        return nextId++;
    }
    finally 
    { 
        Monitor.Exit(locker); 
    }
}

```

#### Bloki ``lock``

Użycie bloku blokady `lock` umożliwia wyłączny dostęp do fragmentu kodu dla pojedynczego wątku. Kompilator rozwija instrukcję `lock` do 
kodu z użyciem klasy `Monitor`.

```csharp
class ThreadSafe 
{
   static object locker = new object();
   static int val1, val2;

   static void Go() 
   {
      lock (locker) // Thread safe!
      {
          if (val2 != 0) Console.WriteLine (val1 / val2);
          val2 = 0;
       }
    }
}

```

#### Obiekty synchronizujące

Dowolny obiekt typu referencyjnego, widoczny dla kooperujących ze sobą wątków, może zostać użyty jako obiekt blokady. 
Obiekt synchronizujący jest zwykle prywatnym polem obiektu lub statycznym polem klasy.

```csharp
class ThreadSafe
{
    List <string> _list = new List <string>();

    void Test()
    {
        lock (_list)
        {
            _list.Add ("Item 1");
            ...

```

#### Stosowanie blokad

Dowolne pole dostępne dla wielu wątków (z których przynajmniej jeden jest wątkiem zmieniającym stan tego pola) powinno zostać umieszczone wewnątrz bloku `lock`. 
Nawet w najprostszym przypadku należy rozważyć synchronizację. Proste typy z .NET Framework są thread-safe tylko podczas współbieżnego odczytu. Zapewnienie całkowitego bezpieczeństwa należy do programisty.
Przykład thread-unsafe:

```csharp
class ThreadUnsafe 
{
   static int x;

   static void Increment() { x++; }
   static void Assign() { x = 123; }
}

```

Modyfikacja do kodu thread-safe:

```csharp
class ThreadUnsafe 
{ 
   static object locker = new object();
   static int x;

   static void Increment() { lock (locker) x++; }
   static void Assign() { lock (locker) x = 123; }
}

```

#### Blokady rekursywne

Blokady mogą być pozyskiwane rekursywnie na tym samym obiekcie synchronizującym.

```csharp
lock(locker)
    lock(locker)
        lock(locker)
        {
            //...
        }

```

Obiekt jest odblokowany, kiedy opuszczony zostanie najbardziej zewnętrzny blok `lock`.
#### Zakleszczenia

Zakleszczenie (*deadlock*)
    sytuacja, w której co najmniej dwa różne wątki czekają na siebie nawzajem, więc żadny nie może się zakończyć.
Do zakleszczenia może dojść w sytuacji, gdy istnieją przynajmniej dwa zasoby i dwa wątki ubiegające się o dostęp do tych zasobów.

```csharp
object locker1 = new object();
object locker2 = new object();

new Thread (() => {
                      lock (locker1)
                      {
                          Thread.Sleep (1000);
                          lock (locker2); // Deadlock
                      }
                  }).Start();
lock (locker2)
{
    Thread.Sleep (1000);
    lock (locker1); // Deadlock
}

```

Aby zminimalizować ryzyko zakleszczenia, należy pozyskiwać blokady zawsze w tej samej kolejności. Implementują kon
### Międzyprocesowe obiekty synchronizujące

Kernel Windowsa posiada własne obiekty umożliwiające synchronizację. Są to:

* muteksy
* semfory
* obiekty zdarzeń

Obiekty te umożliwiają synchronizację między wątkami należącymi do różnych procesów. 
Wszystkie te obiekty mogą być w jednym z dwóch stanów:

* sygnalizowanym - `Signaled`
* niesygnalizowanym - `NonSignaled`
#### Klasa ``WaitHandle``

Klasa `WaitHandle` hermetyzuje uchwyty synchronizujące Win32. Umożliwia obiektom synchronizowanym oczekiwanie oraz sygnalizację między wątkami.
`WaitHandle` jest klasą abstrakcyjną, po której dziedziczą klasy:

* `Mutex`
* `EventWaitHandle` oraz pochodne `AutoResetHandle` i `ManualResetHandle`
* `Semaphore`
#### Muteks

Klasa `Mutex` umożliwia synchronizację dostępu do zasobów, podobnie jak klasa `Monitor` (blok `lock`), z tą różnicą, że muteks może mieć zasięg ogólnosystemowy. Wiele procesów może korzystać z tego samego muteksa synchronizując dostęp do współdzielonych zasobów systemowych.
Tworzenie muteksów:

*  `Mutex()` lub `Mutex(bool initiallyOwned)` – tworzenie muteksów lokalnych
*  `Mutex(bool initiallyOwned, string name, bool createNew)`  – tworzenie muteksów globalnych, które mają unikatową nazwę

Muteks może być:

*  Sygnalizowany – nie posiada właściciela. W takiej sytuacji metoda `WaitOne()` zwraca `true` i wywołujący ją wątek pozyskuje muteks i uzyskuje 

   dostęp do synchronizowanego fragmentu kodu. Aby zwolnić muteks, wątek, który go pozyskał musi wywołać  metodę `RealeaseMutex()`.

*  Niesygnalizowany – jest już pozyskany przez wątek.

Zwolnienie muteksu:

*  `ReleaseMutex()` – jeżeli aplikacja zostanie przerwana CLR zwolni pozyskane muteksy automatycznie

Przykład:

```csharp
static int nextId;
static Mutex mtx = new Mutex();

static int NextId() 
{
    mtx.WaitOne();

    try 
    {
        return nextId++;
    }
    finally 
    { 
        mtx.ReleaseMutex(); 
    }
}

```

#### Semafory

Klasa `Semaphore` dziedziczy po klasie `WaitHandle`. Semafory umożliwiają jednoczesny dostęp do zasobu przez wiele wątków. Liczba wątków jest ograniczona przez określoną maksymalną wartość semafora.
Konstruktor: `public Semaphore(int początkowaWartość, int końcowaWartość)`
Jeśli wątek wywoła metodę `WaitXXX()` semafora:

*  Nie zostanie zablokowany, jeśli wartość semafora jest większa od zera
*  Gdy wątek otrzyma dostęp do kodu, wartość semafora zmniejsza się o jeden
*  Wartość semafora zwiększa się o jeden, gdy wątek wywoła metodę `Release()`

Semafor jest w stanie `Signaled`, kiedy wartość licznika jest większa od zera.
Od wersji .NET Framework 4.0 istnieją dwie implementacje semaforów.

* `Semaphore` - implementacja wykorzystująca obiekty kernela (czas pozyskania ~1us)
* `SemaphoreSlim` - lekka implementacja zoptymalizowana do wymagań programowania współbieżnego (czas pozyskania ~0.25us)
#### Zdarzenia resetujące

Zdarzenia resetujące umożliwiają komunikację między wątkami z wykorzystaniem sygnałów. 
Komunikacja zwykle dotyczy zasobu, do którego wątki potrzebują wyłącznego dostępu. Wątek oczekuje na sygnał przez wywołanie `WaitOne()` – wątek wywołujący jest blokowany. 
Blokada może zostać zwolniona przez inny wątek poprzez wywołanie metody `Set()`.
Dwa typy zdarzeń:

* `AutoResetEvent` - używany dla ekskluzywnego dostępu do zasobu przez wątek (tylko jeden wątek z grupy wątków oczekująch po wywołaniu `WaitOne()` jest wybudzony i otrzymuje dostęp do zasobu)

```csharp
  class BasicWaitHandle
  {
      static EventWaitHandle _waitHandle = new AutoResetEvent (false);

      static void Main()
      {
          new Thread (Waiter).Start();
          Thread.Sleep (1000);                  // Pause for a second...
          _waitHandle.Set();                    // Wake up the Waiter.
      }

      static void Waiter()
      {
          Console.WriteLine ("Waiting...");
          _waitHandle.WaitOne();                // Wait for notification
          Console.WriteLine ("Notified");
      } 
  }

```

* `ManualResetEvent` - używany do komunikacji między wątkami w sytuacji, gdy jeden wątek musi zakończyć swoją pracę i odblokować inne wątki
* `ManualResetEventSlim` - lekka wersja `ManualResetEvent`

```csharp
  static void Main(string[] args)
  {
      var matchers = new[] {"dowjones", "ftse", "nasdaq", "dax"};
      var controlFileAvailable = new ManualResetEventSlim();
      var tasks = new List<Task>();

      foreach (string matcherName in matchers)
      {
          var matcher = new Matcher(matcherName, MatchesFound, controlFileAvailable);
          tasks.Add(matcher.Process());
      }

      Console.WriteLine("Press enter when control file ready");
      Console.ReadLine();

      controlFileAvailable.Set();

      Task.WaitAll(tasks.ToArray());
  }

  private void InternalProcess()
  {
      IEnumerable<TradeDay> days = Initialize();

      controlFileAvailable.Wait();

      ControlParameters parameters = GetControlParameters();
      IEnumerable<TradeDay> matchingDays = null;
      if (parameters != null)
      {
          matchingDays = from d in days
                         where d.Date >= parameters.FromDate &&
                               d.Date <= parameters.ToDate && d.Volume >= parameters.Volume
                         select d;
      }

      matchesFound(dataSource, matchingDays);
  }

```

## Klasa ``CountdownEvent``

Obiekt klasy `CountdownEvent` umożliwia poczekanie ukończenie zadań wykonywanych przez zadaną liczbę wątków. 
CountdownEvent posiada wydajną implementację. 
Konstruktor przyjmuje jako argument wartość licznika. Wywołanie `Signal()` dekrementuje stan licznika, z kolei wywołanie `Wait()` jest 
blokujące dopóki licznik nie osiągnie wartości zero.

```csharp
static CountdownEvent _countdown = new CountdownEvent (3);

static void Main()
{
    new Thread (SaySomething).Start("From thread #1");
    new Thread (SaySomething).Start("From thread #2");
    new Thread (SaySomething).Start("From thread #3");

    _countdown.Wait();   // Blocks until Signal has been called 3 times

    Console.WriteLine ("All threads have finished speaking!");
}

static void SaySomething (object thing)
{
    Thread.Sleep (1000);
    Console.WriteLine (thing);

    _countdown.Signal();
}

```

Istnieje możliwość zwiększenia wartości licznika poprzez wywołanie `AddCount()` (jeżeli wcześniej stan licznika jest zerowy, to rzucony zostanie
wyjątek).
## Klasa ``BackgroundWorker``

Klasa `BackgroundWorker` jest klasą pomocniczą z przestrzeni nazw `System.ComponentModel`  umożliwiającą łatwe zarządzanie wątkiem roboczym.
Cechy BackgroundWorker:

* Flaga "cancel" umożliwiająca przerwanie wątku bez wywołania `Abort()`
* Standardowy protokół raportowania postępu wykonywanego zdania
* Implementuje interfejs `IComponent` – widoczny komponent w przyborniku kontrolek w VS
* Wygodny mechanizm obsługi wyjątków rzucanych z wnętrza wątku
* Umożliwia aktualizację stanu kontrolek Windows Forms (informowanie o postępie pracy wątku roboczego)

Przykład:

```csharp
class Program 
{
   static BackgroundWorker bw = new BackgroundWorker();
   static void Main() 
   {
       bw.DoWork += bw_DoWork;
       bw.RunWorkerAsync ("Message to worker");
       Console.ReadLine(); 
   }   

   static void bw_DoWork (object sender, DoWorkEventArgs e) 
   {
       Console.WriteLine (e.Argument);
       // wykonywanie czasochłonnych operacji...  
   }
}

```

`BackgroundWorker` zapewnia zdarzenie `RunWorkerCompleted` wyzwalane w momencie zakończenia pracy przez wątek roboczy.
Aby obsłużyć raportowanie postępu, należy:

* Ustawić właściwość `WorkerReportsProgress`  na true
* Cyklicznie wywoływać metodę `ReportProgress` z wnętrza metody obsługującej zdarzenie `DoWork` przekazując jako argument liczbę określającą ile procent zadania zostało wykonane
* Obsłużyć zdarzenie `ProgressChanged` – informacja o procencie wykonania znajduje się w argumencie zdarzenia `ProgressPercentage` 
* Kod umieszczony wewnątrz handlera zdarzenia `ProgressChanged` lub `RunWorkerCompleted` może aktualizować stan kontrolek UI

Aby obsłużyć anulowanie zadania, należy:

* Ustawić właściwość `WorkerSupportsCancellation` na true
* Cyklicznie sprawdzać właściwość `CancellationPending` we wnętrzu handlera zdarzenia `DoWork` – jeżeli jest ustawiona na `true` zakończyć pracę metody przez `return`
## Leniwa inicjalizacja

Częstym problemem w środowisku wielowątkowym jest implementacja leniwej inicjalizacji kosztownego obiektu w sposób *thread-safe*.
Ogólnym rozwiązaniem jest zastosowanie bloku `lock` w implementacji gettera właściwości:

```csharp
Expensive _expensive;

public Expensive
{
    get 
    {
        lock(_expensiveLock)
        {
            if (_expensive == null)
                _expensive = new Expensive;

            return _expensive;
        }

    }
}

```

Wydajniejszym rozwiązaniem (stosującym *double check locking pattern*) jest klasa `Lazy<T>`:

```csharp
Lazy<Expensive> _expensive = new Lazy<Expensive> (() => new Expensive(), true);

public Expensive Expensive { get { return _expensive.Value; } }

```

## Thread-Local Storage

Czasami najwygodniejszym sposobem uniknięcia współdzielenia, jest zadbanie o to, aby każdy wątek pracował na własnej, lokalnej dla wątku zmiennej.
Od wersji 4.0 .NET Framework zapewnia wygodny sposób tworzenia zmiennych TLS zarówno dla pól statycznych, jak i instancyjnych, za pomocą klasy `ThreadLocal<T>`.

```csharp
var localRandom = new ThreadLocal<Random>(() => new Random());

```

    Thread thd = new Thread(() => Console.WriteLine(localRandom.Value.Next()));
    thd.Start();
## Timer

`System.Threading.Timer` to prosty timer umożliwiający interwałowe wywoływanie metody w osobnym wątku w puli wątków. Właściwość `Interval` jest typu long.
Aby wyspecyfikować metodę, która ma być regularnie uruchamiana należy użyć delegata `TimerCallback`:

```csharp
void TimerCallback(Object state);

```

Przykład:

```csharp
TimerCallback timerCB = new TimerCallback(IntervalFunction);
Timer copyTimer = new Timer(timerCB, data, 0, 5000); // Interval (ms)

```
