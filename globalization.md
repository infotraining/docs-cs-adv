# Globalizacja aplikacji
## Kultura

Kultura reprezentuje język połączony opcjonalnie z położeniem geograficznym lub politycznym. Użytkownik zinternacjonalizowanej aplikacji preferuje zwykle jedną kulturę, która jest używana do przedstawiania danych w oparciu o regionalne, kulturowe i językowe implikacje takiej preferencji. W przypadku `API` wielokrotnego użytku międzynarodowi użytkownicy oczekują przełączników dla określonych kultur i otrzymywania komunikatów o wyjątkach i dokumentacji w lokalnym języku.
Klasa `System.Globalization.CultureInfo` stanowi podstawę modelu globalizacji `.NET Framework`. Reprezentuje ona instancję konkretnego języka i, opcjonalnie, lokalizacji. Taka kombinacja jest reprezentowana tekstowo przez kod kultury.
Kody kultury są oparte na normach:

* ISO-639: kody reprezentacji nazw języków (*Codes for the representation of names of languages*), pisane małymi literami
* ISO-3166: kody nazw krajów i terytoriów zależnych (*Codes for country and dependent names*), pisane wielkimi literami

Część kodu kultury odnosząca się do kraju jest opcjonalna, ponieważ często aplikacja dostarcza tylko zlokalizowaną część opartą na języku. Rzeczywiste języki używane w regionach geograficznych ograniczają liczbę ważnych permutacji.
W `.NET Framework` istnieje wiele interfejsów `API`, które automatycznie dostosowują swoje działanie do bieżącej kultury użytkownika. Na przykład, operacje konwersji zmiennej na łańcuch i odwrotnie, czyli `ToString` i `Parse`. Można zapisać w bazie danych w formacie `en-US` walutę, daty i czas oraz liczby i napotkać pewne problemy, gdy aplikacja spróbuje automatycznie pracować z tymi informacjami na komputerze w innym kraju.
### Kultura neutralna

Kultura bez preferencji kraju jest nazywana neutralną.
Na przykład, język hiszpański, reprezentowany jest przez kod językowy `ISO-639 es`. Możemy utworzyć `CultureInfo` opierając się na samym kodzie języka `es`.
Drzewo przykładowych kultur neutralnych:
![image](_images/neutral-culture.*)

### Kultura specyficzna

Kultura z językiem i regionem nazywana jest specyficzną.
Aby reprezentować określoną kulturę języka hiszpańskiego w Hiszpanii, powinniśmy połączyć kod języka z kodem kraju, tworząc kod kultury `es-ES`.
Drzewo przykładowych kultur specyficznych:
![image](_images/specific-culture.*)

### Pobieranie określonej kultury

Aby utworzyć instancję `CultureInfo` opierając się na kodzie kultury, należy użyć statycznej metody `CultureInfo.GetCultureInfo()`. Jest to przydatne, kiedy dostosowujemy funkcjonalność aplikacji i zapisujemy preferencje użytkownika w bazie danych. Możemy także odczytać tekst z łańcucha agenta przeglądarki `WWW`. Oznacza to, że mamy surowy łańcuch wartości i tworzymy z niego obiekt `CultureInfo`. Do tej metody przekazujemy łańcuch `<język>-<kod kraju>`, a otrzymujemy w pełni skonfigurowany obiekt `CultureInfo`.
Alternatywnie możemy użyć jednego z konstruktorów klasy `CultureInfo`. Zaletą statycznej metody jest to, że wykorzystuje ona wersję kultury z bufora, zamiast za każdym razem alokować nowy obiekt.
Generowanie instancji `CultureInfo` dla kilku wymienionych powyżej kultur:
```c#
CultureInfo cultureEnNeutral = CultureInfo.GetCultureInfo("en");
CultureInfo cultureEnGb = CultureInfo.GetCultureInfo("en-GB");
CultureInfo cultureEnUs = CultureInfo.GetCultureInfo("en-US");

```

Używając jednego z wywołań `API` `CultureInfo`, które przyjmują kod kultury, jesteśmy odpowiedzialni za jego prawidłowe sformatowanie. W przeciwnym razie, wygenerowany zostanie wyjątek `ArgumentException`. Wyjątek ten zostanie również wygenerowany, jeśli kultura odpowiadająca danemu kodowi nie jest obsługiwana. Jeśli łańcuch został sformatowany prawidłowo, a w dalszym ciągu generowany jest ten wyjątek, oznacza to, że nie został zainstalowany odpowiedni pakiet językowy (zdarza się to często w przypadku języków azjatyckich).
### Wyliczanie dostępnych kultur

Statyczna metoda `CultureInfo.GetCultures()` służy do wyświetlania zestawu kultur zainstalowanych na komputerze. Można to wykorzystać do zachęcenia użytkownika do wybrania obsługiwanej kultury w kliencie lub aplikacji `WWW`. Wystarczy przekazać wartość wyliczeniową `CultureType` do metody, aby określić typ otrzymywanej kultury. Zwracane dane zależą od zainstalowanych w systemie operacyjnym pakietów językowych.

* `CultureTypes.NeutralCultures` – zwraca listę kultur neutralnych obsługiwanych na komputerze
* `CultureTypes.SpecificCultures` – zwraca listę kultur specyficznych w lokalnym systemie
* `CultureTypes.AllCultures` – zwraca sumę zbiorów kultur neutralnych i specyficznych

```c#
CultureInfo[] neutralCultures = CultureInfo.GetCultures(CultureTypes.NeutralCultures);

for (int i = 0; i < neutralCultures.Length; i++)
{
    CultureInfo culture = neutralCultures[i];
    Console.WriteLine("{0}: {1} [{2}]", i, culture.DisplayName, cultureName);
}

```

### Zarządzanie kontekstem kultury

Domyślnie programy dziedziczą preferencje kultury z preferencji systemu operacyjnego zapisanych w profilu bieżącego użytkownika. Można nimi zarządzać za pomocą `Panelu sterowania` `Windows`, z menu `Opcje regionalne i językowe`.
Każdy wątek ma kulturę, która jest używana przez wykonujący go kod. W większości programów klienta ustawienie to nie różni się dla wątków działających wewnątrz programu. Jednak używanie zróżnicowanych kultur w wątkach wewnątrz aplikacji pozwala na uruchomienie niezależnych fragmentów kodu z różnymi ustawieniami kultur, co może być przydatne w aplikacjach pracujących po stronie serwera. W takich przypadkach zwykle obsługiwanych jest wielu użytkowników z różnymi preferencjami kulturowymi, które muszą być rozpoznawane. Wymaganie to można zmienić przez zmianę kultury wątku. Pozwala to użyć domyślnych właściwości formatowania `.NET Framework`, zamiast na przykład dołączać instancję `CultureInfo` do informacji o stanie sesji.
### Kultura formatowania

Kultura formatowania dla bieżącego wątku jest dostępna przez statyczną właściwość `CultureInfo.CurrentCulture` z atrybutem tylko do odczytu. Jest ona także dostępna przez właściwość `Thread.CurrentThread.CurrentCulture`. Właśnie tu większość wywołań `API` platformy `.NET Framework` szuka informacji o kulturze, wykonując zadania formatowania.
```c#
using System.Globalization;
// …
CultureInfo culture = CultureInfo.CurrentCulture;

```

Na przykład metody `ToString` i `Parse` natywnych typów danych używają tej właściwości do odpowiedniej interpretacji tekstowej reprezentacji danych. Wpływa to na sposób formatowania dat, czasu, liczb i waluty, a także na sposób porównywania i zestawiania łańcuchów. Właściwość `CurrentCulture` nie jest jednak używana przez system zasobów do celów obsługi języka i lokalizacji oprogramowania. Zamiast niej używana jest właściwość `CurrentUICulture`, omówiona poniżej.
Kultura formatowania jest ograniczona do globalizacji opartej na regionie, więc powinna być specyficzna. Jeśli dostarczymy tylko neutralną kulturę, mogą powstać niejasności. Jeśli chcemy uzyskać domyślne ustawienie kraju dla danego języka, możemy przekazać kod języka do statycznej metody `CultureInfo.CreateSpecificCulture()`.
```c#
CultureInfo culture = CultureInfo.CreateSpecificCulture("en");

```

### Kultura UI

W każdym wątku istnieje też oddzielna kultura `CultureInfo.CurrentUICulture`. Można ją także uzyskać za pomocą właściwości `Thread.CurrentThread.CurrentUICulture`. Kultura UI jest używana przez system zasobów do pobierania właściwej zlokalizowanej wersji treści. Nie ma ona wpływu na formatowanie regionalne, ale jest używana do tekstu, mediów i wszystkiego, co jest zapisane wewnątrz pakietu zasobów.
W większości przypadków `CurrentCulture` i `CurrentUICulture` zawierają identyczne wartości, chociaż można użyć oddzielnych kultur dla języka i formatowania regionalnego. Jeśli jednak unieważnimy domyślne ustawienia tylko jednej z nich, powinniśmy rozważyć implikacje tej różnicy. Przeważnie ustawienia te powinny być identyczne.
`CurrentUICulture` dotyczy tylko ustawień języka, więc wystarczy podać kulturę neutralną. Dozwolone jest jednak także podanie kultury specyficznej.
### Kultura instalacyjna i domyślna kultura użytkownika

Przy instalowaniu systemu operacyjnego ustalana jest kultura systemowa. Będzie ona używana jako domyślna kultura dla wszystkich wątków, chyba że użytkownik zdecyduje inaczej. Kod zarządzany może ją zbadać za pomocą właściwości `IntalledUICulture` eksponowanej przez
klasę `CultureInfo`. Jest to po prostu wywołanie funkcji `Win32` `GetSystemDefaultLCID()`
opakowane w klasę `CultureInfo`.
Użytkownik może zmienić domyślne ustawienia systemowe. Jest to nazywane domyślną kulturą użytkownika. Jeśli użytkownik ustawi tę kulturę, będzie ona obowiązywała w każdym nowo utworzonym wątku. Zarządzany kod po prostu używa tej domyślnej kultury. Klasa `CultureInfo`
nie eksponuje tych ustawień bezpośrednio, gdyż ma wewnętrzne metody ich pobierania. Możemy jednak wywołać funkcje `GetUserDefaultLCID` i `GetUserDefaultLangID`, aby uzyskać odpowiednio kulturę formatowania i kulturę UI.
### Zmiana wartości kultury

Klasa `System.Threading.Thread` ma także właściwości `CultureInfo` i `CurrentUICulture`. Źródła tych wartości są w rzeczywistości identyczne. Metody pobierania właściwości klasy `CultureInfo` po prostu wywołują właściwości klasy `Thread`, aby pobrać te informacje. Jednak w aplikacjach międzynarodowych powinniśmy zawsze importować klasę `System.Globalization`, więc posiadanie tych informacji w klasie `CultureInfo` jest przydatne. Metody ustawiania tych właściwości spełniają wymagania bezpieczeństwa klasy `ControlThread`, a zatem niezaufany kod nie może ich nadużyć.
Podstawowa różnica między tymi dwoma lokacjami polega jednak na tym, że właściwości klasy `Thread` są ustawialne, a właściwości klasy `CultureInfo` – tylko do odczytu.
Przykład:
Zmiana kultury bieżącego wątku na hiszpańską w Meksyku (`es-MX`)
```c#
CultureInfo mexicanSpanishCi = CultureInfo.GetCultureInfo("es-MX");
Thread.CurrentThread.CurrentCulture = mexicanSpanishCi;
Thread.CurrentThread.CurrentUICulture = mexicanSpanishCi;

```

Wszystkie domyślne funkcje formatowania i lokalizacji będą od tego momentu używać kultury hiszpańskiej w Meksyku.
Jest to przydatne przy ładowaniu preferencji kultury z pliku konfiguracyjnego lub rekordu bazy danych skojarzonego z użytkownikiem. Nie jest to jednak zalecana praktyka. Jeśli wątek zostanie związany z określoną kulturą, utrzymuje ją, dopóki nie zostanie ona jawnie zmieniona. W najgorszym przypadku może to skutkować użyciem jej przez wątki innych użytkowników, na przykład aplikacji WWW. Jeśli zmienimy kulturę w wątku z puli lub jednym wątku finalizacji, prawdopodobnie natkniemy się na szereg dziwnych i niemożliwych do zdiagnozowania problemów.
### Kultura niezmienna

Kultura niezmienna jest szczególną kulturą, używaną zazwyczaj do tuszowania właściwości międzynarodowych. Mieści się ona w hierarchii kultur i jawnie określa brak preferencji kulturowych. Rzadko jest ustawiana jako preferowana kultura, ale często używana w aplikacjach, które nie były testowane pod względem właściwości międzynarodowych. Jeżeli przekażemy ją do metod `Parse` i `ToString`, to wywołania `API` będą korzystać ze swojej logiki domyślnej.
Statyczna właściwość `CultureInfo.InvariantCulture` jest obiektem singletonowym kultury niezmiennej. Kultura ta nie ma swojego kodu.
```c#
CultureInfo invariant = CultureInfo.InvariantCulture;

```

Kultura niezmienna może być przekazana wywołaniom `API`, które oczekują kultury lub właściwości `IFormatProvider`, albo automatycznie wykonującym formatowanie lub wyspecjalizowaną logikę, które chcemy włączyć.
Na przykład metoda `DateTime.ToString()` automatycznie formatuje zwracany łańcuch na podstawie właściwości `CultureInfo.CurrentCulture`. Aby sformatować łańcuch przy użyciu kultury niezmiennej, musimy przekazać ją jako argument metody `ToString()`.
```c#
DateTime now = DateTime.Now;
Console.WriteLine(now.ToString(CultureInfo.InvariantCulture));

```

Dzięki temu takie wywołanie będzie działało identycznie na wszystkich zlokalizowanych platformach. Jeśli gdzieś znajduje się błąd powodujący zamianę pól miesięcy i dni w dacie, na przykład w niestandardowym kodzie analizy daty, międzynarodowy format daty może spowodować awarię. Oczywiście najlepiej nie używać takiego kodu. Czasami jednak nie mamy nad tym kontroli, jeśli na przykład znajduje się on w bibliotece innej firmy.
W pewnych warunkach niebezpiecznie jest akceptować domyślne działanie zależne od kultury. Złośliwi użytkownicy potrafią znaleźć błędy oprogramowania przez prostą zmianę ustawień kultury w swoim systemie Windows lub ręczną zmianę kultury wątku przed wywołaniem kodu. Jeśli lokalizacja nie została wystarczająco przetestowana, można nieumyślnie otworzyć lukę bezpieczeństwa lub niezawodności albo udostępnić oprogramowanie z dużą liczbą trudnych do powtórzenia błędów, zdarzających się tylko międzynarodowym użytkownikom. W samej platformie `.NET Framework` znaleziono dwie takie luki bezpieczeństwa, jedną po udostępnieniu wersji 1.0, a drugą tuż przed udostępnieniem wersji 2.0.
### Formatowanie

Wspólnym typem używanym do reprezentowania specyfikacji formatowania jest interfejs `System.IFormatProvider`. Klasa `CultureInfo`, podobnie jak `DateTimeFormatInfo` i `NumberFormatInfo`, implementuje interfejs `IFormatProvider`. Większość funkcji formatowania polega na tym interfejsie. Inne interfejsy, takie jak `Calendar` dla dat i czasu, są używane przez klasy formatujące w celu wykonania odpowiednich przekształceń.
## Zasoby

Zasób jest dowolną treścią podlegającą zmianom w zależności od kultury. Zasoby są zapisywane w indywidualnych plikach według kultur, a następnie pobierane przez aplikację w czasie jej wykonywania za pomocą wywołań `API` zasobów. Podsystem zasobów dba o lokalizowanie, wczytywanie i pobieranie tych informacji na żądanie, wybierając właściwą wersję na podstawie obowiązującej kultury . Aby to było możliwe, projektant aplikacji musi przenieść wszystkie lokalizowane informacje do pakietów zasobów i utworzyć określone pakiety dla różnych kultur, które chce obsługiwać.
Większość treści zasobu przybiera formę par klucz-wartość, gdzie klucz jest unikatowym identyfikatorem, a wartość – tekstem lub wartością binarną skojarzoną z tym kluczem. Aplikacja wyszukuje tę informację, używając identyfikatora, który jest przekierowany do odpowiedniej zlokalizowanej wersji zasobu na podstawie kultury użytkownika. Ten proces jest zarządzany przed podsystem zasobów używający mechanizmu awaryjnego do zapewnienia, że jeśli treść specyficzna dla ustawień regionalnych nie jest dostępna (całkowicie lub częściowo), w zamian zostanie użyta sensowna wartość domyślna. Te dane mogą być przeznaczone dla etykiety użytkownika, okna komunikatu, tekstu wyjątku i tym podobnych.
### Tworzenie zasobów

Procedura generowania plików z zasobami do pewnego stopnia zależy od formatu osadzanych danych. Istnieją dwa podstawowe typy lokalizowanych danych – tekst i pliki nieprzezroczyste (np. bitmapy). Każdy z nich korzysta z konwencji używania unikatowego identyfikatora dla każdego lokalizowanego fragmentu treści. `Framework SDK` zawiera narzędzie `ResGen.exe`, rozpoznające te dwa formaty i kompilujące je w odpowiedni format pliku zasobu `.resources` w celu wdrożenia.
### Zasoby tekstowe

Zlokalizowane łańcuchy są zapisywane i wykorzystywane przez program przy użyciu unikatowych kluczy opartych na łańcuchach. Można łatwo utworzyć plik zasobu zawierający czysto tekstowe pary klucz-wartość w dowolnym edytorze tekstowym. Narzędzie `ResGen.exe` analizuje proste pary `<klucz> = <wartość>` i rozpoznaje `\n` oraz `\t` odpowiednio jako znaki nowego wiersza i tabulacji. Można również dodawać komentarze, używając średnika lub hash.
```c#
# Przykład pliku zasobów
# Ten cały obszar stanowi komentarz

SomeRandomKey = Witaj
ErrorMessage = Wystąpił błąd.\n\nProszę wysłać e-mail z raportem.

```

Kompilowanie do binarnego pliku `.resources` jest realizowane przez narzędzie `ResGen.exe`. Jeśli powyższy plik ma nazwę `MyResources.txt`, to polecenie kompilacji wygląda następująco:
```console
ResGen.exe MyResources.txt MyResources.resources

```

W rezultacie powstaje plik `MyResources.resources`, który może być osadzony w zestawie.
### Zasoby binarne

Ręczne tworzenie zasobów nie działa dla zasobów binarnych. Narzędzie `ResGen.exe` rozpoznaje oddzielny format `.resx`, który przechowuje pary klucz-wartość w języku `XML`. Ten format pliku obsługuje treść kodowaną binarnie. Można go łatwo utworzyć za pomocą programu `Resource Editor` w `Visual Studio`. Alternatywnie można rozważyć użycie klasy `System.Resources.ResXResourceWriter` umieszczonej w zestawie `System.Windows.Forms.dll`. Pozwala ona na programowe dodanie pliku binarnego do zasobu.
Tworzenie narzędzia `AddResX.exe` upraszczającego dodawanie binarnych zasobów do istniejących plików `.resx`:
```c#
using System;
using System.Collections;
using System.IO;
using System.Resources;

class Program
{
    static void Main(string[] args)
    {
        string resXFile = args[0];
        string resKey = args[1];
        string resValueFile = args[2];

        using (ResXResourceWriter writer = new ResXResourceWriter(resXFile))
        {
            Console.WriteLine("Associating {0} with {1}'s contents", resKey, resValueFile);
            Console.Write("To {0}...", resXFile);

            // Klonowanie istniejącej zawartości:
            using (ResXResourceReader reader = new ResXResourceReader(resXFile))
            {
                foreach (DictionaryEntry node in reader)
                    writer.AddResource((string)node.Key, node.Value);
            }

            // Dodanie nowego klucza:
            writer.AddResource(resKey, File.ReadAllBytes(resValueFile));
        }
        Console.WriteLine("done.");
    }
}

```

Aby dodać bitmapę do istniejącego, zdefiniowanego powyżej zasobu, możemy w prosty sposób przekształcić plik `MyResources.txt` w plik `.resx`.
```console
ResGen.exe MyResources.txt MyResources.resx

```

Spowoduje to utworzenie pliku `XML` zawierającego nasz łańcuch tekstowy.
```xml
<?xml version="1.0" encoding=" utf-8"?>
<root>
<!-- Dla zwięzłości pominięto fragment pliku zawierający XML Scheme -->
    <data name="SomeRandomKey">
        <value xml:space="preserve">Witaj</value>
    </data>
    <data name="ErrorMessage">
        <value xml:space="preserve">Wystąpił błąd. Please send email reporting it.</value>
    </data>
</root>

```

Następnie możemy dodać nowy klucz z dołączonymi binarnymi danymi, korzystając z zaprojektowanego narzędzia.
```console
AddResX.exe MyResources.resx LogoBitmap myLogo.bmp

```

Wynikowy plik `.resx` zawiera teraz zakodowaną w formacie `base-64` bitmapę `myLogo.bmp` skojarzoną z kluczem `LogoBitmap`.
### Pakowanie i wdrażanie

Po wygenerowaniu plików zasobów musimy zapakować je w taki sposób, aby platforma `.NET Framework` mogła je znaleźć w czasie wykonywania. Najczęściej spotykanym modelem jest zestaw satelitarny, który zawiera tylko zasoby dla pojedynczej kultury. Aplikacja musi przechowywać te pliki w strukturze katalogów, bazującej na kulturze.
Jeśli na przykład aplikacja ma zasoby angielskie, niemieckie i polskie, to struktura katalogów wygląda następująco:
![image](_images/catalog-structure.*)

Na rysunku znajdują się trzy zestawy `.NET`. Główną aplikacją jest `YourApp.exe`, do której zostały dołączone zasoby `App.resources`. `App.resources` to domyślny (w tym przypadku angielski) plik zasobów, którego używają kultury bez określonej obsługi. W katalogach `de` i `pl` znajdują się zestawy satelitarne dla języka niemieckiego i polskiego. `App.de.resources` to treść zasobu przetłumaczona na język niemiecki, a `App.pl.resources` – na polski. Używane są one do wygenerowania zestawów `YourApp.xx.dll`, które nie zawierają nic oprócz właściwego znacznika kultury w manifeście zestawu i w informacjach dotyczących zasobu. Do uruchomienia aplikacji nie są wymagane pliki .resources pod warunkiem, że zostały wbudowane, a nie dołączone do zestawów satelitarnych.
Główny plik `App.resources` można połączyć z `YourApp.exe` za pomocą kompilatora języka. Na przykład możemy kompilatorowi `cs.exe` przekazać opcję `/resource`, wyszczególniając plik `App.resources` do wbudowania do głównego zestawu.
Aby wygenerować zestaw satelitarny, należy użyć kompilatora języka lub narzędzia `al.exe`. W celu wygenerowania pliku `YourApp.de.dll` uruchamiamy następujące polecenie w katalogu de aplikacji:
```console
al.exe /t:lib /culture:de /embed:App.de.resources /out:YourApp.de.dll

```

Można także zmienić nazwę zestawu satelitarnego używając różnych opcji polecenia `al.exe`.
### Dostęp do zasobów

Po spakowaniu zasobów musimy uzyskać do nich dostęp z naszego programu. Istnieją dwa ogólne modele obsługiwane przez platformę `.NET Framework`:

* Ze ścisłą kontrolą typów – preferowany
* Ze słabą kontrolą typów – używany przy konserwacji odziedziczonego kodu lub do programowania w wersji 1.x platformy
#### Zasoby ze ścisłą kontrolą typów

Zasoby ze ścisłą kontrolą typów są w rzeczywistości generowane jako kod przez narzędzie `SDK ResGen.exe`. Kompilując tekst lub zasoby `.resx` do pliku `.resources`, możemy dodać opcję `/str:<język>`, a narzędzie wygeneruje plik źródłowy w podanym języku. Obsługiwane są między innymi `C`#, `VB` i `C++`. Otrzymany plik może być kompilowany i używany w naszym programie.
Domyślnie generowana jest klasa typu `internal`. Dodanie opcji `/publicClass` spowoduje wygenerowanie klasy typu `public`.
Klasa wynikowa ma po jednej statycznej właściwości dla każdego klucza zasobu. Właściwości te są dostępne w naszym programie. Hermetyzują one całą logikę wyszukiwania zestawów satelitarnych lub awaryjnego przełączania do zasobów domyślnych.
Możemy na przykład uruchomić plik `.resx` w następujący sposób:
```console
ResGen.exe /str:C# MyResources.resx

```

Wynikowy plik `MyResources.cs` zawiera klasę w przybliżeniu równoważną do:
```c#
using System.Globalization;
using System.Resources;

internal class MyResources
{
    internal static ResourceManager ResourceManager { get; }
    internal static CultureInfo Culture { get; set; }
    internal static string ErrorMessage { get; }
    internal static string SomeRandomKey { get; }
    internal static byte[] LogoBitmap { get; }
}

```

Później możemy wywoływać właściwości, nie martwiąc się, skąd pochodzą informacje. Ten sposób dostępu do zasobów korzysta z tej samej infrastruktury, która nazywana jest "infrastrukturą ze słabą kontrolą typów".
#### Zasoby ze słabą kontrolą typów

Większość informacji o zasobach ze słabą kontrolą typów można znaleźć w automatycznie generowanym pliku klasy zasobów.
Zaczynamy od stworzenia obiektu klasy `System.Resources.ResourceManager` i przekazania mu nazwy bazowej pliku zasobów oraz zestawu służącego jako podstawa do wyszukiwania zestawów satelitarnych.
```c#
ResourceManager rm = new ResourceManager("App", Assembly.GetExecutingAssembly());

```

Następnie szukamy określonych zasobów, używając jednej z metod `GetXxx`:

* `GetString()` – akceptuje klucz zasobu i zwraca ten zasób jako typ `string`
* `GetObject()` – akceptuje klucz i może być użyta do pobrania serializowanych danych typu `byte[]`

Wadą w porównaniu z zasobami z silną kontrolą typów jest to, że przekazywanie kluczy tekstowych jest podatne na błędy typograficzne, co prowadzi do wyjątków `NullReferenceException` w czasie wykonywania.
## Kodowanie

Przy zapisywaniu lub odczytywaniu tekstu używamy kodowania do precyzyjnego określenia bitów reprezentujących znaki. Ostatecznie, gdy dane są zapisywane na dysku lub transmitowane przez sieć, wszystko jest po prostu sekwencją bajtów. W celu przekształcenia tych bajtów w taki sposób, aby miały sens dla użytkownika, potrzebujemy standardowego sposobu ich interpretowania.
Od lat w czysto tekstowych dokumentach stosowane jest kodowanie `ASCII` (`American Standard Code for Information Interchange`) oraz szereg rozszerzonych zestawów znaków. Sam kod `ASCII` korzysta tylko z 7 bitów na znak (z 1 bitem zarezerwowanym na ewentualne rozszerzenia) do przedstawienia 128 możliwych unikatowych znaków. Niestety, 128 znaków nie wystarczy do reprezentowania globalnego zestawu znaków. Większość rozszerzeń `ASCII`, takich jak strony kodowe `ANSI` i `OEM` w Windows, dodaje obsługę znaków łacińskich i innych dla określonej kultury, wykorzystując dodatkowy bit dla górnego zestawu znaków (tj. od 128 do 255). Jednak nie wystarcza to w odniesieniu do wielu języków, zwłaszcza jeśli chcemy wyświetlać na ekranie równocześnie znaki z różnych języków.
W miarę, jak rozpowszechniano oprogramowanie międzynarodowe, wprowadzono 16-bitowe kodowanie znaków, nazywane także 2-bajtowym. Kodowanie `Unicode` stało się standardem. Wykorzystanie pełnych 2 bajtów na znak oznacza, że można użyć w przybliżeniu 65 000 indywidualnych punktów kodu. Jest to nadmierne uproszczenie, ponieważ stosowanie punktów zmiennej długości oraz par punktów kodowych powoduje zwiększenie tej liczby. Szerokie przyjęcie tego kodowania uprościło wymianę zakodowanych w `Unicode` danych pomiędzy aplikacjami i platformami.
Kodowanie `UTF-8`, opisane w dokumencie `RFC 2279`, to odmiana `Unicode`, która zmniejsza rozmiar zakodowanego strumienia. Wykorzystuje ono 8 lub 7 bitów dla dolnych punktów kodu (np. podstawowego zestawu znaków `ASCII`) i zmienną liczbę bajtów dla znaków od kodach wyższych niż 127. Pojedynczy znak Unicode w zakresie od U+0000 do U+FFFF wymaga w `UTF-8` od 1 do 3 bajtów. Maksymalna długość kodowania w `UTF-8` to 6 bajtów, chociaż obecnie nie zdefiniowano jeszcze punktów kodu `Unicode`, które wymagałyby w `UTF-8` więcej niż 4 bajtów. Do pozostałych punktów kodowych wykorzystywane są znaki 16-bitowe. `UTF-7` to odmiana `UTF-8` opisana w dokumencie `RFC 2152`. Format metadanych `CLR` używa `UTF-8` do kodowania większości tekstu, na przykład tablic łańcuchów. Do reprezentowania tekstu w czasie wykonywania używane są pełne znaki `Unicode` (tj. `wchar_t`).
`ASCII`, `Unicode`, `UTF-8` i `UTF-7` to najczęściej spotykane sposoby kodowania. Istnieje wiele innych sposobów kodowania. Konwersja na rozmaite systemy kodowania i z nich w czasie wykonywania jest łatwa dzięki różnym klasom `Encoding`.
### Obsługa BCL

Typ `System.Text.Encoding` jest klasą abstrakcyjną, z której wywodzi się zestaw klas konkretnych `Encoding`:

* Encoding.ASCII – obsługuje tradycyjne 7-bitowe kodowanie ASCII (punkty kodowe od U+0000 do U+007F)
* Encoding.Default – równoważna ASCII i uzyskiwana przez wywołanie Win32 GetACP
* Encoding.Unicode – domyślnie z kolejnością bajtów typu młodszy-starszy
* Encoding.BigEndianUnicode – używana do współpracy z niektórymi platformami sprzętowymi
* Encoding.UTF8
* Encoding.UTF7
* Encoding.UTF32

Oprócz statycznych właściwości możemy uzyskać instancję klasy `Encoding` przekazując nazwę lub numer strony kodowej `Windows`.
Tworzenie instancji klasy `Encoding` dla języków łacińskich (strona kodowa 1252):
```c#
Encoding westernEuropean = Encoding.GetEncoding(1252);

```

Klasa `Encoding` zawiera kilka interesujących składowych. Przekazując zmienną `CLR` typu `string` lub `char[]` do `GetBytes()`, uzyskamy docelowo zakodowane bajty. Różne typy w przestrzeni nazw `System.IO` korzystają z tego mechanizmu do serializacji zawartości strumienia zgodnie z wybranym kodowaniem. Odwrotnie działa składowa `GetChars()`. Podając zmienną typu `byte[]`, uzyskamy przeanalizowaną zawartość w zmiennej `char[]`. Typy te zależą od typów `Encoder` i `Decoder`.
#### Integracja wejścia-wyjścia

Kodowanie jest stosowane głównie przy wykonywaniu operacji wejścia-wyjścia. Wiele wywołań `API` klas `File`, `Stream` oraz klas odczytu i zapisu w przestrzeni `System.IO` pozwala na przekazywanie instancji klasy `Encoding` w celu wskazania sposobu odczytu lub zapisu danych. Jeśli kodowanie było stosowane przy tworzeniu danych tekstowych, musi być także używane przy ich odczycie. W przeciwnym razie bajty zostaną zinterpretowane nieprawidłowo, powodując błędy.
#### Znaczniki kolejności bajtów

W wielu przypadkach klasy `System.IO` domyślnie wybierają właściwe kodowanie dzięki korzystaniu ze znacznika kolejności bajtów BOM (`byte-order-mark`). Jest to 16-bitowa sygnatura (U+FEFF) na początku pliku zakodowanego w `Unicode`, która służy do wskazania kolejności bajtów (czy czytnik powinien zobaczyć bajty w kolejności 0xFE 0xFF, czy też 0xFF 0xFE). Znacznik ten jest również używany do oznaczenia kodowania `UTF-8` i `UTF-7`. Konstruktory klasy `System.IO.StreamReader` mają parametr `detectEncodingFromByteOrderMarks` służący do określenia, czy znaczniki kolejności bajtów powinny być automatycznie wykrywane i uwzględniane. Ma on domyślną wartość `true`.
## Problemy z domyślną kulturą

Platforma `.NET Framework` została zaprojektowana do obsługi automatycznego formatowania dla określonej kultury w wielu interfejsach `API`. W większości przypadków jest to dobre rozwiązanie. Na przykład, przy wdrożeniu w innej kulturze daty będą formatowane prawidłowo bez prowadzania zmian w kodzie. Sortowanie i porównywanie łańcuchów znakowych, które automatycznie różni się w zależności od kultury, także jest wygodne. Może jednak powodować problemy – od łagodnych błędów, które zdarzają się tylko na międzynarodowych platformach, po poważne luki w zabezpieczeniach platformy. Problemy pierwszego typu występują znacznie częściej niż drugiego.
### Manipulacja łańcuchami

Kanonicznym przykładem zależnych od kultury wywołań `API` są metody przekształcania prostego typu danych w jego reprezentację łańcuchową i odwrotnie – analizowanie łańcucha i przekształcanie go w prosty typ danych. Służą do tego funkcje `ToString()`,` Parse()` i `TryParse()`, dostępne dla wszystkich prostych typów danych, takich jak `Int32`(C# `int`), `Int64`(C# `long`), `Double`(C# `double`), `DateTime` i `Decimal` (C# `decimal`).
Proste typy danych oferują także przeciążone wersje funkcji `ToString`, `Parse` i `TryParse`, które akceptują parametr `IFormatProvider`. Parametr ten jest wykorzystywany do przekazania parametrów formatowania, co pozwala na przekazanie precyzyjnej instancji kultury `XxxFormatInfo` w celu zmodyfikowania domyślnego działania.
Domyślne implementacje tych metod jako `IFormatProvider` używają właściwości `CultureInfo.CurrentCulture`. Oznacza to, że domyślnie uzyskamy działanie, które zmienia się w zależności od kultury wykonywanego wątku.
Przykład:
Jeśli będziemy analizować listę liczb z pliku tekstowego, zadanie to może się nieoczekiwanie nie powieść, gdy włączymy inną kulturę.
```c#
string numberString = "33,200.50";
double number = double.Parse(numberString);

```

Ten fragment kodu w angielskiej kulturze działa bez problemu. Spróbujmy jednak wykonać go po następującym wierszu kodu.
```c#
Thread.CurrentThread.CurrentCulture = CultureInfo.GetCultureInfo("de-DE");

```

Może się to stać niejawnie na podstawie profilu bieżącego użytkownika Windows lub bez naszej wiedzy w kodzie, który napisał inny programista. Zostanie zgłoszony wyjątek `FormatException` lub zostanie zwrócona wartość `false`, jeśli użyliśmy funkcji `TryParse()`. Dzieje się tak dlatego, że `de-DE` reprezentuje kulturę niemiecką, w której do oddzielenie dziesiętnych części liczby jest używany znak przecinka, a nie kropki, zaś do oddzielenia grup liczb – kropka, a nie przecinek. W języku niemieckim liczba 33,200.50 nie ma sensu. Jeśli oryginalny łańcuch będzie zawierał liczbę 200.50, aplikacja pomyślnie przeanalizuje liczbą, ale będzie ona znaczyła dwadzieścia tysięcy pięćdziesiąt zamiast dwustu i pięćdziesięciu setnych.
Rozwiązaniem tego problemu jest `CultureInfo.InvariantCulture`. Jeśli tekstowa reprezentacja liczby lub innego typu danych jest serializowana przy użyciu `CultureInfo.InvariantCulture`, to zostanie prawidłowo odtworzona.
Zmodyfikujmy powyższy fragment kodu:
```c#
string numberString = "33,200.500";
double number = double.Parse(numberString, CultureInfo.InvariantCulture);

```

Niezależnie od kultury wybranej w `Panelu sterowania` użytkownika, kod będzie działał w sposób przewidywalny.
Funkcja `ToString()` powoduje podobne problemy. W zależności od sposobu użycia łańcucha, możemy potrzebować wersji uwzględniającej kulturę lub nie. Jest to także użyteczne w powyższym przykładzie. Chociaż analizujemy łańcuch używając `InvariantCulture`, możemy po prostu wywołać funkcję `ToString()`, aby uzyskać prawidłową reprezentację na platformie niemieckiej. Możemy także przekazać parametr `InvariantCulture` do funkcji `ToString()`, aby zignorować bieżącą kulturę. Jeśli pracujemy z `ADO.NET`, `XML` lub interfejsami `API` usług WWW, to zawsze działają one z reprezentacją niezmienną.
Testowanie prawidłowości operacji specyficznych dla kultury jest skomplikowane, ponieważ `.NET Framework` wykonuje wiele działań niejawnie. W miarę możliwości powinniśmy pracować z danymi opierając się na `InvariantCulture`. Należy rozważyć jawne przekazywanie `CultureInfo.CurrentCulture` jako parametru do tych wywołań `API`.
#### Porównywanie i sortowanie łańcuchów

Istnieje zestaw funkcji związanych z operacjami na łańcuchach, których działanie zmienia się w zależności od bieżącej kultury. Porównywanie dwóch łańcuchów pokazuje różnice między kulturami na skutek różnych reguł sortowania i zmiany wielkości znaków. Metoda `String.Equals()` oraz przeciążone operatory `op_Equality` (==) i `op_Inequality` (!=) nie są czułe na kulturę. Metoda `Equals()` po prostu porównuje dwa łańcuchy w poszukiwaniu czysto leksykalnej równoważności (porównanie porządkowe), co oznacza, że nie próbuje wykonywać porównań określonych kulturowo. Podobnie wywołanie `String.CompareOrdinal` lub `Equals(string, string, StringComparison)` z wartością `StringComparison.Ordinal` wykonuje porządkowanie na podstawie wartości znaku w łańcuchu, a nie alfabetycznie.
#### Kolejność porównywania i sortowania

Statyczna metoda `Compare()` i instancyjna metoda `CompareTo()` zależą jednak od kultury. Używają natywnego porządkowania alfabetycznego opartego na języku kultury. Metoda `Compare()` operuje na dwóch łańcuchach a i b, a metoda `CompareTo()` – this i b.
Metody te zwracają:

* <0, jeśli a znajduje się przed b
* 0, jeśli a i b są równe
* >0 – jeśli a znajduje się po b

Wywołanie metody `Equals` z wartością `StringComparison.CurrentCulture` porównuje łańcuchy w ten sam sposób.
Metody te są odpowiednie w sytuacji, gdy algorytmy sortujące muszą porównać i uporządkować listę łańcuchów. Użycie kultury do określenia tego porządku często ma sens. Jednak oznacza to, że właściwości sortowania mogą być inne na różnych platformach. Instancyjna metoda `CompareTo` nie oferuje przeciążenia przyjmującego parametr `IFormatProvider`. Do tych celów należy użyć statycznej właściwości `String`. `Compare` lub lepiej `CompareOrdinal`.
Dostępnych jest kilka funkcji do sortowania listy obiektów `String`. Klasy `System.Collection.SortedList` i `System.Collection.Generic.SortedList<TKey,TValue>` sortują swoje wewnętrzne tablice. Wykorzystują w tym celu domyślną implementację metody `IComparable.CompareTo()` lub `IComparable<T>.CompareTo()`. W przypadku obiektów `String` jest to wspomniana wyżej, uwzględniająca kulturę metoda `CompareTo()`. Podobnie metody `BinarySearch()` i `Sort()` klasy `System.Array` domyślnie używają metody `CompareTo()`. We wszystkich tych przypadkach sortowanie zmienia się w zależności od kultury. Wszystkie te metody mają wersje przeciążone, które przyjmują instancję `IComparer` w celu niestandardowych porównań, co można wykorzystać do uzyskania niezmiennego sposobu sortowania. Wystarczy przekazać statyczną właściwość `Comparer.DefaultInvariant`, a do wszystkich porównań używana będzie kultura niezmienna.
```c#
string[] strings = /*...*/;

// Można określić DefaultInvariant dla SortedList:
SortedList list = new SortedList(Comparer.DefaultInvariant);
foreach (string s in strings) list.Add(s, null);

// Alternatywnie można posortować tablicę w miejscu używając DefaultInvariant:
Array.Sort(strings, Comparer.DefaultInvariant);

```

#### Zmiana wielkości liter przy porównywaniu i sortowaniu

Wiele programów normalizuje łańcuchy, przekształcając je na małe lub wielkie litery. W większości przypadków wynik tej operacji jest następnie używany do porównania niewrażliwego na wielkość znaków. Jednak w niektórych alfabetach – a zwłaszcza tureckim – znaki łacińskie są pomieszane z narodowymi i reguły zmiany wielkości liter prowadzą do konfliktu. Ten problem "tureckiego I" bierze się stąd, że w alfabecie tureckim używane są znaki i oraz I, ale nie są one ze sobą związane. 'i' (U+0069) ma oddzielny znak wielkiej litery – 'İ' (U+0130), a 'I' (U+0049) ma osobny znak małej litery – 'ı' (U+0131).
```c#
CultureInfo[] cultures = new CultureInfo[] {
    CultureInfo.GetCultureInfo("en-US"),
    CultureInfo.GetCultureInfo("tr-TR")
};

char lower = 'i';
char upper = 'I';

foreach (CultureInfo culture in cultures)
{
    Thread.CurrentThread.CurrentCulture = culture;
    Console.WriteLine("{0}", culture.DisplayName);

    // Wykonuje konwersję z małej litery i na wielką
    char toUpper = Char.ToUpper(lower);
    Console.WriteLine(" Lower->Upper: {0} ({1:X}) -> {2} ({3:X})", 
                      lower, (int)lower, toUpper, (int)toUpper);

    // Wykonuje konwersję z wielkiej litery I na małą
    char toLower = Char.ToLower(upper);
    Console.WriteLine(" Upper->Lower: {0} ({1:X}) -> {2} ({3:X})", 
                      upper, (int)upper, toLower, (int)toLower);
}

```

Po uruchomieniu programu otrzymamy:
```console
English (United States)
    Lower->Upper: i (69) -> I (49)
    Upper->Lower: I (49) -> i (69)

Turkish (Turkey)
    Lower->Upper: i (69) -> İ (130)
    Upper->Lower: I (49) -> ı (131)

```

Przejście od małej do wielkiej litery i oraz z powrotem prowadzi do całkiem innego znaku ı. Jest to powód oczywistych problemów, jeśli normalizujemy tekst w celu wykazania równości lub wykonania porównania. Problem może być poważniejszy, jeśli wykonujemy oparte na tekście porównanie, aby wymusić sprawdzenie bezpieczeństwa. Na przykład, przekazujemy adresy `URL` do pewnego systemu w celu pobrania danych i potrzebujemy gwarancji, że użytkownicy nie będą mogli pytać o pliki na dysku.
```c#
void DoSomething(string path)
{
    if (String.Compare(path, 0, "FILE:", 0, 5, true) == 0)
        throw new SecurityException("Nie można tego wykonać!");
    // Wykonanie operacji
}

```

Jeśli użytkownik używa kultury tureckiej, może przekazać `URL` `file://etc/` i skłonić metodę `DoSomething()` do wykonania jakiejś operacji na pliku dyskowym. Stanie się tak, ponieważ łańcuch po normalizacji zostanie przekształcony na `FİLE://ETC/` na skutek wspomnianych problemów ze zmianą wielkości znaków.
Metody `ToUpper()` i `ToLower()` klas `String` i `Char` domyślnie uwzględniają kulturę. Istnieją wersje `ToUpperInvariant()` i `ToLowerInvariant()`, które niejawnie używają `InvariantCulture`. Metody uwzględniające kulturę mają także wersje przeciążone, które akceptują parametr `IFormatProvider`, co oznacza, że można do nich dostarczyć precyzyjną informację `CultureInfo`, jeśli potrzebne jest specjalne traktowanie wielkości znaków.
