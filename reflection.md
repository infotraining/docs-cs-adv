# Refleksje

Przeglądanie metadanych i skompilowanego kodu w czasie wykonywania nazywa się refleksją.
`API` refleksji jest zawarte w przestrzeni `System.Reflection`.
## Refleksje typów
### Uzyskiwanie informacji o typie

Instancja `System.Type` reprezentuje metadane dla typu. Ponieważ `Type` jest powszechnie używany, więc znajduje się w przestrzeni nazw `System`, a nie w `System.Reflection`.
Aby otrzymać instancję `System.Type`, należy wywołać `GetType()` na dowolnym obiekcie lub użycia operatora C# `typeof`.

```csharp
Type t1 = DateTime.Now.GetType();           // Type obtained at runtime
Type t2 = typeof(DateTime);                 // Type obtained at compile time

```

Aby otrzymać typy tablicowe i typy generyczne, można użyć `typeof`.

```csharp
Type t3 = typeof(DateTime[]);               // 1-d Array type
Type t4 = typeof(DateTime[,]);              // 2-d Array type
Type t5 = typeof(Dictionary<int,int>);      // Closed generic type
Type t6 = typeof(Dictionary<,>);            // Unbound generic type

```

Można również pobrać `Type` po nazwie. Jeśli znamy referencję do jego zestawu, wystarczy wywołać `Assembly.GetType()`.

```csharp
Type t = Assembly.GetExecutingAssembly().GetType("Demos.TestProgram");

```

Jeśli nie mamy obiektu `Assembly`, możemy pobrać typ korzystając z kwalifikowanej nazwy zestawu (pełna nazwa typu i pełna kwalifikowana nazwa zestawu).
Zestaw ładowany jest niejawnie, jak gdybyśmy wywołali `Assembly.Load(string)`.

```csharp
Type t = Type.GetType("System.Int32, mscorlib, Version=2.0.0.0, Culture=neutral,
PublicKeyToken=b77a5c561934e089");

```

Mając obiekt `System.Type` możemy użyć jego właściwości, aby uzyskać dostęp do nazwy, zestawu, typu bazowego, widzialności i innych cech typu.

```csharp
Type stringType = typeof (String);
string name = stringType.Name;              // String
Type baseType = stringType.BaseType;        // typeof(Object)
Assembly assem = stringType.Assembly;       // mscorlib.dll
bool isPublic = stringType.IsPublic;        // true

```

Instancja `System.Type` udostępnia wszystkie metadane dla typu oraz zestawu, w którym został zdefiniowany.
`System.Type` jest klasą abstrakcyjną, więc operator `typeof` musi w rzeczywistości zwracać instancję klasy pochodnej po `Type`. Klasa pochodna używana przez `CLR` jest klasą internal w zestawie `mscorlib` i nazywana jest `RuntimeType`.
#### Pobieranie typów zagnieżdżonych

Aby pobrać typy zagnieżdżone, należy wywołać `GetNestedTypes()` na typie zawierającym.

```csharp
foreach (Type t in typeof(System.Environment).GetNestedTypes())
    Console.WriteLine(t.FullName);

```

```console
System.Environment+SpecialFolder

```

Jedynym ograniczeniem dla typów zagnieżdżonych jest fakt, że `CLR` traktuje je jakby miały specjalne, zagnieżdżone poziomy dostępu.

```csharp
Type t = typeof(System.Environment.SpecialFolder);
Console.WriteLine(t.IsPublic);              // False
Console.WriteLine(t.IsNestedPublic);        // True

```

### Nazwy typów

Typ posiada właściwości `Namespace`, `Name` i `FullName`. W większości przypadków `FullName` składa się z `Namespace` i `Name`.

```csharp
Type t = typeof(System.Text.StringBuilder);
Console.WriteLine(t.Namespace);             // System.Text
Console.WriteLine(t.Name);                  // StringBuilder
Console.WriteLine(t.FullName);              // System.Text.StringBuilder

```

Istnieją dwa odstępstwa od tej reguły:

* Zamknięte typy generyczne
* Typy zagnieżdżone
#### Nazwy typu zagnieżdżonego

Dla typów zagnieżdżonych, typ zawierający pojawia się tylko w `FullName`:

```csharp
Type t = typeof  (System.Environment.SpecialFolder);

Console.WriteLine (t.Namespace);              // System
Console.WriteLine (t.Name);                   // SpecialFolder
Console.WriteLine (t.FullName);               // System.Environment+SpecialFolder

```

Symbol "+" odróżnia typ zawierający od zagnieżdżonej przestrzeni nazw.
#### Nazwy typu generycznego

Nazwy typu generycznego poprzedzone są symbolem ', a zakończone liczbą parametrów typu. 
Jeśli typ generyczny jest niezwiązany, ta zasada odnosi się zarówno do `Name`, jak i do `FullName`.

```csharp
Type t = typeof(Dictionary<,>);               // Unbound
Console.WriteLine(t.Name);                    // Dictionary'2
Console.WriteLine(t.FullName);                // System.Collections.Generic.Dictionary'2

```

Jeśli typ generyczny jest zamknięty, `FullName` zawiera przyrostek. Dla każdego parametru typu wypisywana jest pełna kwalifikowana nazwa zestawu.

```csharp
Console.WriteLine (typeof (Dictionary<int,string>).FullName);

```

```console
System.Collections.Generic.Dictionary'2[
[System.Int32, mscorlib, Version=2.0.0.0, Culture=neutral, 
PublicKeyToken=b77a5c561934e089], 
[System.String, mscorlib, Version=2.0.0.0, Culture=neutral, 
PublicKeyToken=b77a5c561934e089]]

```

Dzięki temu `AssemblyQualifiedName` (połączenie pełnej nazwy typu i nazwy zestawu) zawiera informacje wystarczające do pełnej identyfikacji zarówno typu generycznego, jak i parametrów tego typu.
### Typy bazowe i interfejsy

`Type` udostępnia właściwość `BaseType`.

```csharp
Type base1 = typeof(System.String).BaseType;
Type base2 = typeof(System.IO.FileStream).BaseType;
Console.WriteLine(base1.Name);                                // Object
Console.WriteLine(base2.Name);                                // Stream

```

Metoda `GetInterfaces()` zwraca interfejsy implementowane przez typ.

```csharp
foreach (Type iType in typeof(Guid).GetInterfaces())
    Console.WriteLine(iType.Name);

```

```console
IFormattable
IComparable
IComparable'1
IEquatable'1  

```

Refleksja udostępnia dwa dynamiczne odpowiedniki statycznego operatora (`is`):

* `IsInstanceOfType()` – przyjmuje typ i instancję
* `IsAssignableFrom()` – przyjmuje dwa typy

Przykład użycia `IsInstanceOfType()`:

```csharp
object obj = Guid.NewGuid();
Type target = typeof (IFormattable);
bool isTrue = obj is IFormattable;                            // Statyczny operator C#
bool alsoTrue = target.IsInstanceOfType(obj);                 // Odpowiednik dynamiczny

```

`IsAssignableFrom()` jest bardziej wszechstronna:

```csharp
Type target = typeof (IComparable), source = typeof (string);
Console.WriteLine (target.IsAssignableFrom(source));          // True

```

Metoda `IsSubclassOf()` działa na tych samych zasadach, co metoda `IsAssignableFrom()`, ale wyklucza interfejsy.
### Tworzenie instancji typów

Istnieją dwie możliwości dynamicznego tworzenia instancji obiektu z jego typu:

* Wywołanie statycznej metody `Activator.CreateInstance()`
* Wywołanie `Invoke()` na obiekcie `ConstructorInfo` otrzymanym z wywołania `GetConstructor()` na `Type` (zaawansowane scenariusze)

`Activator.CreateInstance()` przyjmuje `Type` i argumenty opcjonalne, które są przekazywane do konstruktora.

```csharp
int i = (int)Activator.CreateInstance(typeof(int));
DateTime dt = (DateTime)Activator.CreateInstance(typeof(DateTime), 2000, 1, 1);

```

`CreateInstance()` pozwala podać wiele innych opcji:

* Zestaw, z którego ma być ładowany typ
* Docelowa domena aplikacji
* Możliwość wiązania z konstruktorem nonpublic (wyjątek `MissingMethodException` zostanie rzucony, jeśli nie ma pasującego konstruktora)

Wywołanie `Invoke()` dla `ConstructorInfo()` jest konieczne, jeśli wartości argumentu nie mogą dokonać rozróżnienia pomiędzy przeciążonymi konstruktorami.
Załóżmy, że klasa `X` ma dwa konstruktory – pierwszy przyjmujący parametr typu `string`, a drugi – typu `StringBuilder`. Jeśli przekażemy argument `null` do `Activator.CreateInstance()`, docelowy konstruktor jest niejednoznaczny. W takiej sytuacji należy użyć `ConstructorInfo`.

```csharp
// Przechwytujemy konstruktor, który przyjmuje pojedynczy parametr typu string
ConstructorInfo ci = typeof(X).GetConstructor(new[] { typeof (string) });

// Konstruujemy obiekt używając metody przeciążonej i przekazując null
object foo = ci.Invoke(new object[] { null });

```

### Typy generyczne

`Type` może reprezentować zamknięty lub niezwiązany typ generyczny. W czasie kompilacji można utworzyć instancję zamkniętego typu generycznego, lecz nie ma takiej możliwości dla typu niezwiązanego.

```csharp
Type closed = typeof(List<int>);
List<int> list = (List<int>)Activator.CreateInstance(closed); // OK

Type unbound = typeof(List<>);
object anError = Activator.CreateInstance(unbound);           // Błąd czasu wykonania

```

Metoda `MakeGenericType()` konwertuje niezwiązany typ na zamknięty typ generyczny. Wystarczy przekazać argumenty typu.

```csharp
Type unbound = typeof(List<>);
Type closed = unbound.MakeGenericType(typeof (int));

```

Metoda `GetGenericTypeDefinition()` działa odwrotnie.

```csharp
Type unbound2 = closed.GetGenericTypeDefinition();            // unbound == unbound2

```

Właściwość `IsGenericType` zwraca `true`, jeśli `Type` jest typem generycznym.
Metoda `IsGenericTypeDefinition()` zwraca `true`, jeśli typ generyczny jest niezwiązany.
Poniższy kod służy do sprawdzenia, czy typ jest typem z dopuszczalną wartością pustą:

```csharp
Type nullable = typeof(bool?);
Console.WriteLine(nullable.IsGenericType && 
                  nullable.GetGenericTypeDefinition() == typeof(Nullable<>));   // True

```

`GetGenericArguments()` zwraca argumenty typu dla zamkniętych typów generycznych.

```csharp

```

  Console.WriteLine(closed.GetGenericArguments()[0]);           // System.Int32
  Console.WriteLine(nullable.GetGenericArguments()[0]);         // System.Boolean
Dla niezwiązanych typów generycznych, metoda `GetGenericArguments()` zwraca pseudotypy, które reprezentują parametry wymienione w definicji typu generycznego.

```csharp

```

  Console.WriteLine(unbound.GetGenericArguments()[0]);          // T
W czasie wykonania wszystkie typy generyczne są albo niezwiązane, albo zamknięte.
Niezwiązane są w przypadku (stosunkowo rzadkim) wyrażenia takiego jak `typeof(Foo<>)`. W pozostałych przypadkach są zamknięte. Typy generyczne nie mogą być otwarte w czasie wykonania. Wszystkie otwarte typy są zamykane przez kompilator.
Metoda w poniższej klasie zawsze wyświetla `False`:

```csharp
class Foo<T>
{
    public void Test()
    {
        Console.Write(GetType().IsGenericTypeDefinition);
    }
}

```

## Refleksja składowych

Metoda `GetMembers` zwraca składowe typu.
Rozważmy poniższą klasę:

```csharp
sourceclass CoffeeMachine
{
    private bool isOn;
    public void On()
    {
        isOn = true;
    }
    public void Off()
    {
        isOn = false;
    }
}

```

Refleksja publicznych składowych:

```csharp
MemberInfo[] members = typeof(CoffeeMachine).GetMembers();
foreach (MemberInfo m in members)
    Console.WriteLine (m);

```

Wynik:
```console
Void On()
Void Off()
System.String ToString()
Boolean Equals(System.Object)
Int32 GetHashCode()
System.Type GetType()
Void .ctor()

```

Metoda `GetMembers()` wywołana bez argumentów zwraca wszystkie składowe publiczne dla typu (i jego typów bazowych). `GetMember()` pobiera konkretną składową po nazwie, ale zwraca tablicę, ponieważ składowe mogą być przeciążone.

```csharp
members = typeof(CoffeeMachine).GetMember("On");
Console.WriteLine(members[0]);                                // Void On()

```

`MemberInfo` posiada również właściwość `MemberType` typu `MemberTypes`. Jest to flagowy typ wyliczeniowy z poniższymi wartościami:

| All         | Custom | Field  | NestedType | TypeInfo |
|-------------|--------|--------|------------|----------|
| Constructor | Event  | Method | Property   |          |

Wywołując `GetMembers()` można przekazać instancję `MemberTypes` w celu ograniczenia rodzajów zwracanych składowych. Można również ograniczyć zbiór wynikowy poprzez wywołanie `GetMethods()`, `GetFields()`, `GetProperties()`, `GetEvents()`, `GetConstructors()` lub `GetNestedTypes()`. Istnieją również wersje pojedyncze wymienionych metod, które mogą być używane dla konkretnej składowej.
Obiekt `MemberInfo` posiada właściwość `Name` i dwie właściwości `Type`:

* `DeclaringType` – zwraca `Type` definiujący składową
* `ReflectedType` – zwraca `Type`, dla którego została wywołana metoda `GetMembers`
### Typy składowej

`MemberInfo` jest abstrakcyjną klasą bazową dla typów wymienionych poniżej.
![image](_images/member-info.*)

Można rzutować `MemberInfo` na jego podtypy w oparciu o właściwość `MemberType`. Jeśli składowa została otrzymana z użyciem `GetMethod()`, `GetField()`, `GetProperty()`, `GetEvent()`, `GetConstructor()` lub `GetNestedType()` (lub ich wersji mnogich), rzutowanie nie jest potrzebne.

Metody używane dla konstrukcji C#:

| Konstrukcja C#     | Metoda              | Nazwa                   | Wynik                                                                            |
|--------------------|---------------------|-------------------------|----------------------------------------------------------------------------------|
| Metoda             | GetMethod()         | Nazwa metody            | MethodInfo                                                                       |
| Właściwość         | GetProperty()       | Nazwa właściwości       | PropertyInfo                                                                     |
| Indekser           | GetDefaultMembers() |                         | MemberInfo[] (zawierający obiekty PropertyInfo, jeśli zostały skompilowane w C#) |
| Pole               | GetField()          | Nazwa pola              | FieldInfo                                                                        |
| Składowa wyliczana | GetField()          | Nazwa składowej         | FieldInfo                                                                        |
| Zdarzenie          | GetEvent()          | Nazwa zdarzenia         | EventInfo                                                                        |
| Konstruktor        | GetConstructor()    |                         | ConstructorInfo                                                                  |
| Finalizer          | GetMethod()         | "Finalize"              | MethodInfo                                                                       |
| Operator           | GetMethod()         | "op_" + nazwa operatora | MethodInfo                                                                       |
| Typ zagnieżdżony   | GetNestedType()     | Nazwa typu              | Type                                                                             |

Każda klasa pochodna po `MemberInfo` ma do dyspozycji właściwości i metody udostępniające wszystkie aspekty metadanych klasy składowej. Obejmuje to widoczność, modyfikatory, argumenty typu generycznego, parametry, zwracany typ i atrybuty użytkownika.
Przykład użycia `GetMethod()`:

```csharp
MethodInfo m = typeof(CoffeeMachine).GetMethod("On");
Console.WriteLine(m);                                         // Void On()
Console.WriteLine(m.ReturnType);                              // System.Void

```

Wszystkie instancje `*Info` są cachowane przez `API` refleksji przy pierwszym użyciu.

```csharp
MethodInfo method = typeof(CoffeeMachine).GetMethod("On");
MemberInfo member = typeof(CoffeeMachine).GetMember("On")[0];
Console.Write(method == member);                              // True

```

Poza zachowaniem tożsamości obiektu, caching poprawia wydajność `API`, które w innym razie działa dość wolno.
### Dynamiczne wywołanie składowej

Mając obiekt `MemberInfo`, możemy go dynamicznie wywoływać lub pobierać/ustawiać jego wartości. Jest to nazywane dynamicznym wiązaniem lub późnym wiązaniem, ponieważ wybieramy składową do wywołania w czasie wykonania, a nie w czasie kompilacji.
W poniższym przykładzie użyto zwykłego wiązania statycznego:

```csharp
string s = "Hello";
int length = s.Length;

```

To samo można wykonać dynamicznie z użyciem refleksji:

```csharp
object s = "Hello";
PropertyInfo prop = s.GetType().GetProperty ("Length");
int length = (int)prop.GetValue(s, null);                     // 5

```

`GetValue()` i `SetValue()` pobierają i ustawiają wartość `PropertyInfo` lub `FieldInfo`.
Pierwszy argument to instancja, która może być null lub składową statyczną. Dostęp do indeksera odbywa się na tej samej zasadzie, co dostęp do właściwości Item z tą różnicą, że musimy podać wartości indeksera jako drugi argument przy wywołaniu `GetValue()` lub `SetValue()`.
Aby dynamicznie wywołać metodę, należy wywołać `Invoke()` na instancji `MethodInfo`, podając tablicę argumentów, które mają zostać przekazane do tej metody. W przypadku błędnego typu argumentu zostanie rzucony wyjątek w czasie wykonania. Stosując dynamiczne wywołanie tracimy bezpieczeństwo typu w czasie kompilacji, ale nadal mamy bezpieczeństwo w czasie wykonania (tak jak w przypadku słowa kluczowego `dynamic`).
### Parametry metod

Załóżmy, że chcemy dynamicznie wywołać metodę `Substring()` dla instancji typu `string`.
Statyczne wywołanie:

```csharp
Console.WriteLine("stamp".Substring(2));                      // "amp"

```

Dynamiczny odpowiednik z użyciem refleksji:

```csharp
Type type = typeof(string);
Type[] parameterTypes = { typeof (int) };
MethodInfo method = type.GetMethod("Substring", parameterTypes);

object[] arguments = { 2 };
object returnValue = method.Invoke("stamp", arguments);
Console.WriteLine(returnValue);                               // "amp"

```

Ponieważ metoda `Substring()` jest przeciążona, więc musimy przekazać tablicę typów parametrowych do `GetMethod()`, aby wskazać interesującą nas wersję. Bez typów parametrowych `GetMethod()` rzuci `AmbiguousMatchException`.
### Dostęp do składowych niepublicznych

Wszystkie metody stosowane na typach w celu odczytania metadanych (np. `GetProperty()`, `GetField()`, itp.) mają swoje wersje przeciążone, które przyjmują wartość wyliczeniową `BindingFlags`. Działa ona jak filtr metadanych i pozwala zmieniać domyślne kryteria wyboru. Najczęstszym zastosowaniem jest pobranie niepublicznych składowych.
Rozważmy poniższą klasę:

```csharp
class CoffeeMachine
{
    private bool isOn;
    public void On()
    {
        isOn = true;
    }
    public void Off()
    {
        isOn = false;
    }
    public override string ToString()
    {
        return string.Format("CoffeeMachine is " + (isOn ? "on" : "off"));
    }
}

```

Możemy zmienić stan instancji z pominięciem metod `On()` i `Off()`:

```csharp
Type t = typeof(CoffeeMachine);
CoffeeMachine cm = new CoffeeMachine();
cm.On();
FieldInfo f = t.GetField("isOn", BindingFlags.NonPublic | BindingFlags.Instance);
f.SetValue(cm, false);
Console.WriteLine(cm);                                        // CoffeeMachine is off

```

Używanie refleksji w celu dostępu do niepublicznych składowych ma wiele zalet, ale jest również niebezpieczne, ponieważ możemy pominąć enkapsulację tworząc niezarządzalną zależność od wewnętrznej implementacji typu.
### Sterowanie wiązaniem

Wartość wyliczenia bitowego `BindingFlags` może być wykorzystana w operacjach bitowych do szybkiego sprawdzenia typu wiązania:
```console
BindingFlags.Public | BindingFlags.Instance
BindingFlags.Public | BindingFlags.Static
BindingFlags.NonPublic | BindingFlags.Instance
BindingFlags.NonPublic | BindingFlags.Static

```

`NonPublic` zawiera `internal`, `protected`, `protected internal` i `private`.
Pobieranie wszystkich publicznych, statycznych składowych obiektu typu:

```csharp
BindingFlags publicStatic = BindingFlags.Public | BindingFlags.Static;
MemberInfo[] members = typeof (object).GetMembers(publicStatic);

```

Pobieranie wszystkich niepublicznych składowych obiektu typu, zarówno statycznych jak i instancyjnych:

```csharp
BindingFlags nonPublicBinding =
    BindingFlags.NonPublic | BindingFlags.Static | BindingFlags.Instance;
MemberInfo[] members = typeof(object).GetMembers(nonPublicBinding);

```

Flaga `DeclaredOnly` wyklucza funkcje dziedziczone z typów bazowych chyba, że zostały nadpisane.
### Metody generyczne

Metody generyczne nie mogą być wywoływane bezpośrednio. Poniższy kod spowoduje rzucenie wyjątku.

```csharp
class Program
{
    public static T Echo<T> (T x) { return x; }
    static void Main()
    {
        MethodInfo echo = typeof(Program).GetMethod ("Echo");
        Console.WriteLine (echo.IsGenericMethodDefinition);         // True
        echo.Invoke (null, new object[] { 123 } );                  // Wyjątek
    }
}

```

Wymagany jest dodatkowy krok, w którym należy wywołać `MakeGenericMethod()` na instancji `MethodInfo`, określając argumenty typu generycznego. Zwracana jest kolejna instancja `MethodInfo`, która może zostać wywołana w poniższy sposób.

```csharp
MethodInfo echo = typeof (Program).GetMethod ("Echo");
MethodInfo intEcho = echo.MakeGenericMethod (typeof (int));
Console.WriteLine (intEcho.IsGenericMethodDefinition);          // False
Console.WriteLine (intEcho.Invoke (null, new object[] { 3 } )); // 3

```

## Informacje o zestawie

Można dynamicznie pobrać informacje o zestawie wywołując `GetType()` lub `GetTypes()` na obiekcie `Assembly`.
Za pomocą poniższego kodu pobieramy z bieżącego zestawu typ `TestProgram` z przestrzeni nazw `Demos`:

```csharp
Type t = Assembly.GetExecutingAssembly().GetType("Demos.TestProgram");

```

Przykład:
Lista wszystkich typów w zestawie `mylib.dll` w `e:\\demo` 

```csharp
Assembly a = Assembly.LoadFrom(@"e:\demo\mylib.dll");
foreach (Type t in a.GetTypes())
    Console.WriteLine(t);

```

`GetTypes` zwraca listę typów z pominięciem typów zagnieżdżonych.
### Kontekst wczytywania tylko do refleksji

W poprzednim przykładzie załadowaliśmy zestaw do bieżącej domeny aplikacji w celu wypisania jego typów. To może spowodować niepożądane efekty uboczne takie jak wykonanie statycznych konstruktorów. Jeśli musimy przeglądać informacje o typie (a nie tworzyć jego instancji lub wywoływać go), rozwiązaniem jest załadowanie zestawu tylko do refleksji (`reflection-only context`).

```csharp
Assembly a = Assembly.ReflectionOnlyLoadFrom(@"e:\demo\mylib.dll");
Console.WriteLine(a.ReflectionOnly);                            // True
foreach (Type t in a.GetTypes())
    Console.WriteLine(t);

```

To jest punkt wyjścia do zaprojektowania przeglądarki klas.
Dostępne są trzy metody ładowania zestawu tylko do refleksji:

* `ReflectionOnlyLoad(byte[])`
* `ReflectionOnlyLoad(string)`
* `ReflectionOnlyLoadFrom(string)`
## Praca z atrybutami

`CLR` pozwala dołączać dodatkowe metadane do typów, składowych i zestawów za pośrednictwem atrybutów. Za pomocą tego mechanizmu jest zarządzanych wiele funkcji `CLR`, takich jak serializacja i bezpieczeństwo.
Kluczową cechą atrybutów jest możliwość tworzenia własnych atrybutów i używania ich do "dekorowania" składnika kodu dodatkowymi informacjami. Te dodatkowe informacje są wkompilowane do zestawu i mogą zostać pobrane w czasie wykonania za pomocą refleksji. Jest to przydatne do tworzenia usług działających deklaratywnie, takich jak automatyczne testy jednostkowe.
### Rodzaje atrybutów

Istnieją trzy rodzaje atrybutów:

* Atrybuty mapowane bitowo
* Atrybuty niestandardowe
* Atrybuty pseudocustom

Z wymienionych rodzajów tylko atrybuty niestandardowe są rozszerzalne.
Większość modyfikatorów `C#`, takich jak `public`, `abstract` i `sealed`, kompiluje się do atrybutów mapowanych bitowo.
Wszystkie atrybuty niestandardowe są reprezentowane przez podklasę klasy `System.Attribute`.
Atrybuty pseudocustom wyglądają i działają tak jak zwykłe atrybuty niestandardowe. Są reprezentowane przez podklasę klasy `System.Attribute` i dołączane w zwykły sposób.

```csharp
[Serializable] public class Foo {...}

```

Różnica polega na tym, że kompilator lub `CLR` optymalizuje wewnętrznie atrybuty pseudocustom konwertując je na atrybuty mapowane bitowo.
### Atrybut AttributeUsage

`AttributeUsage` jest atrybutem stosowanym do klas atrybutowych. Informuje kompilator o sposobie użycia atrybutu docelowego.

```csharp
public sealed class AttributeUsageAttribute : Attribute
{
    public AttributeUsageAttribute (AttributeTargets validOn);
    public bool AllowMultiple { get; set; }
    public bool Inherited { get; set; }
    public AttributeTargets ValidOn { get; }
}

```

`AllowMultiple` sprawdza, czy definiowany atrybut może zostać zastosowany więcej niż raz do tego samego celu.
`Inherited` sprawdza, czy atrybut może posiadać typy podrzędne.
`ValidOn` wskazuje zestaw celów (klasy, interfejsy, właściwości, metody, parametry, itp.), do których można dołączyć atrybut. Przyjmuje dowolną kombinację wartości z typu wyliczeniowego `AttributeTargets`, który posiada następujące składowe:

| All | Delegate | GenericParameter | Parameter |
|-----|----------|------------------|-----------|
| Assembly | Enum | Interface | Property |
| Class | Event | Method | ReturnValue |
| Constructor | Field | Module | Struct |

Użycie `AttributeUsage` dla atrybutu `Serializable`:

```csharp
{[AttributeUsage (AttributeTargets.Delegate | 
                  AttributeTargets.Enum | 
                  AttributeTargets.Struct | 
                  AttributeTargets.Class, Inherited = false)
]
public sealed class SerializableAttribute : Attribute
{
}

```

To jest, w zasadzie, pełna definicja atrybutu `Serializable`. Na tym przykładzie widać, jak proste jest napisanie klasy atrybutowej bez właściwości lub specjalnych konstruktorów.
### Definiowanie własnego atrybutu

Aby napisać własny atrybut:
1. Dziedziczymy po klasie `System.Attribute` lub pochodnej po `System.Attribute`. Konwencja zaleca, żeby nazwa klasy kończyła się słowem "`Attribute`", ale nie jest to wymagane.
2. Stosujemy atrybut `AttributeUsage`. Jeśli atrybut nie wymaga żadnych właściwości czy argumentów, to przechodzimy do następnego kroku.
3. Tworzymy jeden lub więcej publicznych konstruktorów. Parametry konstruktora definiują pozycyjne parametry atrybutu i staną się obowiązkowe przy użyciu atrybutu.
4. Deklarujemy publiczne pole lub właściwość dla każdego nazwanego parametru, który chcemy obsługiwać.
Parametry nazwane są opcjonalne, jeśli używamy atrybutu.
Poniższa klasa definiuje atrybut do obsługi systemu automatycznych testów jednostkowych. Wskazuje metodę, która powinna zostać przetestowana, określa liczbę powtórzeń testu oraz wiadomość wyświetlaną w razie niepowodzenia testu.

```csharp
[AttributeUsage (AttributeTargets.Method)]
public sealed class TestAttribute : Attribute
{
    public int Repetitions;
    public string FailureMessage;

    public TestAttribute () : this (1) { }
    public TestAttribute (int repetitions) { Repetitions = repetitions; }
}

```

Klasa `Foo` z metodami udekorowanymi na rożne sposoby w użyciem atrybutu `Test`:

```csharp
class Foo
{
[Test]
public void Method1() { ... }

[Test(20)]
public void Method2() { ... }

[Test(20, FailureMessage="Debugging Time!")]
public void Method3() { ... }
}

```

### Pobieranie atrybutów w czasie wykonania

Istnieją dwie możliwości pobrania atrybutów w czasie wykonania:

* Wywołanie `GetCustomAttributes()` na dowolnym `Type` lub obiekcie `MemberInfo`
* Wywołanie `Attribute.GetCustomAttribute()` lub `Attribute.GetCustomAttributes()`. Metody te są przeciążone, aby akceptowały dowolny obiekt refleksji, który odpowiada ważnemu celowi atrybutu (`Type`, `Assembly`, `Module`, `MemberInfo` lub `ParameterInfo`).

Lista metod klasy `Foo`, które mają `TestAttribute`:

```csharp
foreach (MethodInfo mi in typeof(Foo).GetMethods())
{
TestAttribute att = 
    (TestAttribute)Attribute.GetCustomAttribute(mi, typeof (TestAttribute));

if (att != null)
    Console.WriteLine("Method {0} will be tested; reps={1}; msg={2}", 
                      mi.Name, att.Repetitions, att.FailureMessage);
}

```

Wynik:
```console

```

  Method Method1 will be tested; reps=1; msg=
  Method Method2 will be tested; reps=20; msg=
  Method Method3 will be tested; reps=20; msg=Debugging Time!
Aby zastosować ten przykład w systemie testów jednostkowych, należy rozszerzyć go o wywołanie metod udekorowanych atrybutem `Test`:

```csharp
foreach (MethodInfo mi in typeof (Foo).GetMethods())
{
    TestAttribute att = 
        (TestAttribute) Attribute.GetCustomAttribute(mi, typeof (TestAttribute));

if (att != null)
    for (int i = 0; i < att.Repetitions; i++)
        try
        {
            // Wywołanie metody bez argumentów
            mi.Invoke (new Foo(), null);
        }
        catch (Exception ex)
        {
            // Opakowanie wyjątku w att.FailureMessage
            throw new Exception ("Error: " + att.FailureMessage, ex);
        }
}

```

Przykład:
Lista atrybutów dostępnych dla danego typu

```csharp
[Serializable, Obsolete]
class Test
{
    static void Main()
    {
        object[] atts = Attribute.GetCustomAttributes (typeof (Test));
        foreach (object att in atts)
            Console.WriteLine (att);
    }
}

```

Wynik:
```console
System.ObsoleteAttribute
System.SerializableAttribute

```
