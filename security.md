# Bezpieczeństwo w .NET Framework
## Uprawnienia

.NET Framework używa uprawnień do implementacji sandbox'ingu lub autoryzacji. **Uprawnienie** działa jak bramka, która chroni przez wykonaniem określonego kodu (potencjalnie niebezpiecznego).

* Uprawnienie to prawo dostępu do zasobu lub do wykonania pewnych operacji.
* Jest reprezentowane w czasie wykonania programu przez obiekt dziedziczący po `System.Security.IPermission`

Obiekt uprawnienia (np. `FileIOPermission`) umożliwia:

* weryfikację, czy dana funkcja oraz funkcje wywołujące (na stosie) mają prawa do wykonania określonej operacji (`Demand()`)
* weryfikację, czy funkcja bezpośrednio wywołująca ma prawo wykonać operacje (`LinkDemand()`)
* tymczasowo uciec z sandbox'a poprzez asercję uprawnień (`Assert()`)
### Rodzaje uprawnień

#### Podstawowe uprawnienia

| Typ                   | Umożliwia                                                              |
|-----------------------|------------------------------------------------------------------------|
| SecurityPermission    | Zaawansowane operacje, w tym prawa do wywoływania kodu niezarządzanego |
| ReflectionPermission  | Używanie refleksji                                                     |
| EnvironmentPermission | Odczyt/ zapis ustawień środowiska linii komend                         |
| RegistryPermission    | Odczyt/ zapis w rejestrze Windows                                      |

#### Uprawnienia I/O

| Typ                              | Umożliwia                                                                          |
|----------------------------------|------------------------------------------------------------------------------------|
| FileIOPermission                 | Odczyt/ zapis plików i katalogów                                                   |
| FileDialogPermission             | Dostęp do plików wskazanych przez użytkownika w oknie dialogowym Otwórz lub Zapisz |
| IsolatedStorageFilePermission    | Dostęp do pamięci izolowanej                                                       |
| ConfigurationPermission          | Odczyt plików konfiguracyjnych aplikacji                                           |
| SqlClientPermission, OleDbPermission, OdbcPermission | Komunikację z serwerem bazy danych za pomocą klasy SqlClient, OleDb lub Odbc       |
| DistributedTransactionPermission | Uczestnictwo w transakcjach rozproszonych                                          |

#### Uprawnienia sieciowe

| Typ                          | Umożliwia                                            |
|------------------------------|------------------------------------------------------|
| DnsPermission                | Uzyskiwanie nazw komputerów z użyciem DNS            |
| WebPermission                | Ustanawianie lub otrzymywanie połączeń internetowych |
| SocketPermission             | Komunikowanie się poprzez sieć z użyciem portów      |
| SmtpPermission               | Wysyłanie wiadomości za pomocą bibliotek SMTP        |
| NetworkInformationPermission | Użycie klas takich jak Ping i NetworkInterface       |

#### Uprawnienia interfejsu użytkownika

| Typ                  | Umożliwia                                                                           |
|----------------------|-------------------------------------------------------------------------------------|
| UIPermission         | Dostęp do funkcjonalności interfejsów użytkownika, np. tworzenie formularzy Windows |
| WebBrowserPermission | Używanie kontrolki WebBrowser                                                       |
| MediaPermission      | Obsługę grafiki, dźwięku i wideo w WPF                                              |
| PrintingPermission   | Dostęp do drukarki                                                                  |

#### Uprawnienia diagnostyczne

| Typ                          | Umożliwia                                                 |
|------------------------------|-----------------------------------------------------------|
| EventLogPermission           | Tworzenie wpisów w systemowym dzienniku zdarzeń           |
| PerformanceCounterPermission | Odczyt/ zapis liczników wydajności na poziomie systemowym |

### Zabezpieczenia imperatywne

* Żądanie, asercja lub odmowa jest zdefiniowana w kodzie metody
* Kod imperatywny używa bezpośrednio obiektów `IPermission` oraz metod `Demand()`, `Assert()`, `Deny()`
* Zaletą zabezpieczeń imperatywnych jest to, że mogą zgłaszać żądania na podstawie informacji dynamicznych

```csharp
void SecureFunction()
{
    FileIOPermission p = new FileIOPermission(
        FileIOPermissionAccess.Read, @"c:\temp\");
    p.Demand();
    // wykonanie chronionej operacji
}

```

### Zabezpieczenia deklaratywne

* Można oznaczać metody, konstruktory, klasy oraz zestawy atrybutami

  - umożliwia to wczesne wykrycie przez CLR żądanych uprawnień
  - może zwiększyć wydajność

```csharp
[UIPermission(SecurityAction.Demand, Window=UIPermissionWindow.AllWindows)]
public Form FindForm()
{
  ...
}

```

### Zbiory uprawnień

* Logiczna grupa uprawnień, które można łącznie przyznać lub odebrać w jakimś interesującym scenariuszu
* Obejmuje nie tylko listę uprawnień, ale również zestaw opcji dotyczących każdego uprawnienia

```csharp
PermissionSet ps = new PermissionSet (PermissionState.None);

ps.AddPermission (new UIPermission (PermissionState.Unrestricted));
ps.AddPermission (new SecurityPermission (
                      SecurityPermissionFlag.UnmanagedCode));
ps.AddPermission (new FileIOPermission (
                      FileIOPermissionAccess.Read, @"c:\docs"));
ps.Demand();

```

## Code Access Security
### Stosowanie CAS

* Plik wykonywalny.NET uruchamiany z powłoki Windows lub linii komend ma nieograniczone uprawnienia, czyli jest w **pełni zaufany** (*fully trusted*).
* Jeśli uruchamiamy zestaw poprzez inne środowisko hostingowe, takie jak host integracji z CLR w SQL Server, ASP.NET, ClickOnce lub niestandardowy host, to host decyduje o przyznaniu uprawnień. Jeśli te uprawnienia są ograniczone, to mówimy o **ograniczonym zaufaniu** (*partially trusted*) lub o stosowaniu **sandboxingu**.
* Host tworzy domenę aplikacji z ograniczonymi uprawnieniami i ładuje zestaw do tak utworzonego sandbox'a

  - inne zestawy ładowane do tej domeny aplikacji (np. zestawy, do których się odwołujemy) działają w tym samym sandbox'ie, z tym samym zbiorem uprawnień

* Istnieją dwa wyjątki od tej reguły:

  - Zestawy zarejestrowane w GAC
  - Zestawy mianowane przez host jako w pełni zaufane
  Zestawy z tych dwóch kategorii są uważane za zestawy w pełni zaufane i mogą uciec z sandbox'a poprzez dokonanie asercji wymaganych uprawnień.
#### Testowanie pełnego zaufania

Sprawdzanie, czy przyznane są nieograniczone uprawnienia (zestaw jest w pełni zaufany):

```csharp
new PermissionSet(PermissionState.Unrestricted).Demand();

```

Jeśli domena aplikacji jest sandbox'em, zostanie rzucony wyjątek.    
#### Atrybuty APTCA i [SecurityCritical]

* Aby zapobiec atakom podniesienia uprawnień, CLR domyślnie nie pozwala częściowo zaufanym zestawom wywoływać zestawów w pełni zaufanych.
* W celu dopuszczenia takich wywołań należy:

  - zastosować atrybut `[AllowPartiallyTrustedCallers]`  - w skrócie APTCA
  - zastosować atrybut `[SecurityTransparent]`

* Przed wersją CLR 4.0 obsługiwany był tylko atrybut APTCA, który dopuszczał częściowo zaufanych wywołujących
* Od wersji CLR 4.0 atrybut APTCA może również niejawnie oznaczać wszystkie metody i funkcje w zestawie jako przezroczyste
* Atrybut `[SecurityTransparent]` stosuje  silniejszą wersję tej samej reguły. 

  - Różnica – stosując APTCA możemy mianować wybrane metody w zestawie jako nieprzezroczyste, natomiast atrybut `[SecurityTransparent]` powoduje, że wszystkie metody są przezroczyste
#### Model przezroczystości w CLR 4.0

* Model przezroczystości bezpieczeństwa ułatwia zabezpieczanie zestawów, które mogą być w pełni zaufane i wywoływane z kodu częściowo zaufanego
* W modelu przezroczystości metody należą do jednej z trzech kategorii:

  - krytyczne (security-critical)
  - krytyczne zaufane (security-safe-critical)
  - żadne z powyższych (w tym przypadku, metody są nazywane przezroczystymi)

* **Metody przezroczyste** (*security transparent*) nie mogą:

  - uruchamiać kodu `unsafe`
  - uruchamiać kodu poprzez P/Invoke lub COM.
  - dokonywać asercji uprawnień w celu podniesienia ich poziomu bezpieczeństwa.
  - spełniać warunku `LinkDemand()`, ponieważ w kodzie przezroczystym wymaganie łącza automatycznie zmienia się w wymaganie pełnego uprawnienia
  - wywoływać metod w .NET Framework oznaczonych jako `[SecurityCritical]`

* **Metody krytyczne** (*security critical*) dla bezpieczeństwa oznaczone są atrybutem `[SecurityCritical]`

  - `[SecurityCritical]` oznacza, że metoda pozwoli uciec częściowo zaufanemu wywołującemu z sandbox'a

*  **Zaufane metody krytyczne** (*security safe critical*) - atrybut `[SecuritySafeCritical]` oznacza, że metoda wykonuje operacje krytyczne pod względem bezpieczeństwa, ale przy odpowiednich zabezpieczeniach jest bezpieczna dla częściowo zaufanych wywołujących.

   - Zaufane metody krytyczne działają jako strażnicy dla metod krytycznych. Mogą być wywoływane przez dowolną metodę w dowolnym zestawie (w pełni zaufanym lub częściowo zaufanym, podlegającym żądaniom CAS opartych na uprawnieniach)

* **Częściowo zaufane zestawy nie mogą wywoływać metod krytycznych z w pełni zaufanych zestawów.**
* Metody `[SecurityCritical]` mogą być wywoływane jedynie przez:

  - inne metody `[SecurityCritical]`
  - metody oznaczone jako `[SecuritySafeCritical]`
Przykład:
Metoda `WatchTV()`, którą będziemy wywoływać musi wywołać `GetTVRoomKey()`, która jest metodą krytyczną.
Musimy oznaczyć metodę `WatchTV()` jako zaufaną metodę krytyczną i zweryfikować odpowiedni zestaw uprawnień.
Opakowujemy metodę krytyczną i w ten sposób czynimy ją bezpieczną dla każdego wywołującego.

```csharp
[SecurityCritical]
public Key GetTVRoomKey() { ... }

[SecuritySafeCritical]
public void WatchTV()
{
    new TVPermission().Demand();
    using (Key key = GetTVRoomKey())
        PrisonGuard.OpenDoor(key);
}

```

#### Przezroczysty kod

* Metody przezroczyste są tak nazywane, ponieważ można je zignorować w czasie sprawdzania kodu pod kątem ataku podniesienia uprawnień
* Należy skupić się na metodach [SecuritySafeCritical], które zwykle stanowią ułamek metod zestawu
* Przezroczyste zestawy nie wymagają sprawdzania pod kątem ataku podniesienia uprawnień i niejawnie dopuszczają częściowo zaufanych wywołujących – nie musimy stosować APTCA
* Istnieją dwa sposoby określenia przezroczystości na poziomie zestawu:

  - zastosowanie APTCA – wszystkie metody będą niejawnie przezroczyste, z wyjątkiem tych oznaczonych inaczej
  - zastosowanie atrybutu zestawu `[SecurityTransparent]` – wszystkie metody, bez wyjątku, będą niejawnie przezroczyste

```csharp
    [assembly: SecurityTransparent]

```

* Można też nie określać przezroczystości dla zestawu, co prowadzi do niejawnego ustawienia opcji `[SecurityCritical]` dla każdej metody w zestawie.

  - wyjątkiem są wirtualne metody [SecuritySafeCritical] , które nadpisujemy - te metody pozostaną safe-critical
  W rezultacie możemy wywołać dowolną metodę (jeśli zestaw jest w pełni zaufany), ale przezroczyste metody z innych zestawów nie mogą wywołać tych metod.
## Hashing

* Hashing danych umożliwia jednokierunkowe szyfrowanie

  - łatwo można wygenerować wartość hash dla danej wiadomości
  - trudno jest zmienić wiadomość bez zmiany wartości hash
  - trudno jest znaleźć dwie wiadomości posiadające tą samą wartość hash

* Wartości hash są często stosowane do przechowywania haseł w bazie danych, ponieważ zwykle nie potrzebujemy (lub nie chcemy) wyświetlać ich odkodowanej wersji. W celu autentykacji, hasło wprowadzone przez użytkownika jest haszowane i porównywane z hashem hasła, który jest przechowany w bazie danych
* Skrót ma zawsze mały, stały rozmiar niezależnie od długości danych źródłowych. 

  - Dzięki temu nadaje się do porównywania plików lub wykrywania błędów w strumieni danych (lepiej niż suma kontrolna)
  - Zmiana pojedynczego bitu w dowolnym miejscu w danych źródłowych powoduje znaczącą zmianę skrótu danych

* .NET Framework oferuję hashowanie za pomocą algorytmów MD5 oraz rodziny algorytmów SHA (*Secure Hash*).
* Główne algorytmy, w porządku rosnącego bezpieczeństwa (i długości tokenu klucza publicznego w bajtach)

  - MD5(16) → SHA1(20) → SHA256(32) → SHA384(48) → SHA512(64)
  - Im krótszy algorytm, tym szybciej się wykonuje
  - Im krótszy skrót, tym większe prawdopodobieństwo kolizji (dwa różne pliki otrzymają ten sam skrót)
  - Dłuższe algorytmy SHA są odpowiednie do haszowania haseł, ale wymagają używania silnych haseł w celu ochrony przed atakiem słownikowym
### MD5

* Zaprojektowany przez Ron Rivest'a w 1991 (następca po MD4)
* Obliczany skrót ma długość 128 bitów (16 bajtów)
* Szeroko stosowany do weryfikacji spójności plików
* W 1996 znaleziono problem w odporności algorytmu na kolizje - w rezultacie zalecane jest stosowanie algorytmów SHA
* MD5 jest ponad 20 razy szybszy niż SHA512 i nadaje się do obliczania sum kontrolnych plików. MD5 umożliwia haszowanie setek megabajtów na sekundę. Wynik jest zapisywany w Guid. Guid ma długość dokładnie 16 bajtów i jako typ wartościowy jest łatwiejszy do obróbki niż tablica bajtów
### SHA

Rodzina algorytmów opublikowanych przez NIST

* SHA-1 - algorytm liczący 160 bitowy skrót (po 2010 roku nie jest już zalecany)
* SHA-2 - rodzina dwóch algorytmów różniących się długością skrótu

  - SHA-256
  - SHA-512
### Obliczanie wartości hash w .NET Framework

Klasa bazowa `HashAlgorithm` oferuje interfejs obliczania skrótów. Dziedziczą po nim klasy MD5, SHA256 oraz SHA512.
W celu haszowania wywołujemy `ComputeHash()` na jednej z klas pochodnych do `HashAlgorithm` takich jak `SHA256` lub `MD5`:

```csharp
byte[] hash;

using (Stream fs = File.OpenRead("checkme.doc"))
    hash = MD5.Create().ComputeHash(fs); // hash ma długość 16 bajtów

```

Metoda `ComputeHash()` może także przyjmować tablicę bajtów, co jest przydatne do haszowania haseł:

```csharp
byte[] data = System.Text.Encoding.UTF8.GetBytes("stRhong%pword");
byte[] hash = SHA256.Create().ComputeHash (data);

```

## Szyfrowanie symetryczne

* Szyfrowanie symetryczne używa tego samego klucza do szyfrowania i odszyfrowywania
* .NET Framework udostępnia cztery algorytmy symetryczne, z których najlepszy jest Rijndael (wymawiany "Rhine Dahl" lub "Rain Doll")
### Algorytmy Rijandael

* Rodzina szybkich i bezpiecznych umożliwiających szyfrowanie z kluczami oraz blokami o rożnej długości.
* Ma dwie implementacje: `Rijandael` oraz `Aes`

  - Jedyną różnicą pomiędzy tymi klasami jest brak możliwości osłabienia szyfru w `Aes` poprzez zmianę rozmiaru bloku

* Klasa `Aes` jest zalecana przez zespół programistów odpowiedzialny za implementację algorytmów bezpieczeństwa CLR
* Klasy `Rijndael` i `Aes` akceptują symetryczne klucze długości 16, 24 lub 32 bajtów. Wszystkie są aktualnie uważane za bezpieczne.
#### Szyfrowanie danych

Szyfrowanie ciągu bajtów tak, jak zostały zapisane w pliku, z użyciem klucza o długości 16 bajtów:

```csharp
byte[] key = {145,12,32,245,98,132,98,214,6,77,131,44,221,3,9,50};
byte[] iv = {15,122,132,5,93,198,44,31,9,39,241,49,250,188,80,7};

byte[] data = { 1, 2, 3, 4, 5 }; // To, co szyfrujemy

using (SymmetricAlgorithm algorithm = Aes.Create())
using (ICryptoTransform encryptor = algorithm.CreateEncryptor(key, iv))
using (Stream f = File.Create("encrypted.bin"))
using (Stream c = new CryptoStream(f, encryptor, CryptoStreamMode.Write))
    c.Write (data, 0, data.Length);

```

Poniższy kod odszyfruje plik:

```csharp
byte[] key = {145,12,32,245,98,132,98,214,6,77,131,44,221,3,9,50};
byte[] iv = {15,122,132,5,93,198,44,31,9,39,241,49,250,188,80,7};

byte[] decrypted = new byte[5];

using (SymmetricAlgorithm algorithm = Aes.Create())
using (ICryptoTransform decryptor = algorithm.CreateDecryptor (key, iv))
using (Stream f = File.OpenRead ("encrypted.bin"))
using (Stream c = new CryptoStream (f, decryptor, CryptoStreamMode.Read))
{
    while(true)
    {
        int b = c.ReadByte();

        if (n <= -1)
            break;

        Console.Write(b + " ");
    }
}

```

#### Klucz i IV

W przykładzie stworzyliśmy klucz z 16 losowo wybranych bajtów. Jeśli przy dekodowaniu zostanie użyty niewłaściwy klucz, `CryptoStream` rzuci wyjątek  `CryptographicException`. Przechwycenie wyjątku jest jedynym sposobem sprawdzenia, czy klucz jest prawidłowy.
Oprócz klucza stworzyliśmy IV (*Initialization Vector*). Ta 16-bitowa sekwencja jest częścią szyfru, podobnie jak klucz, ale nie jest tajna. Przy wysyłaniu zaszyfrowanej wiadomości, IV zostanie przesłany otwartym tekstem (prawdopodobnie w nagłówku wiadomości), ale zmieniamy go w każdej nowej wiadomości. To spowoduje, że żadna z zaszyfrowanych wiadomości nie będzie przypominać poprzednich wiadomości nawet, jeśli ich niezaszyfrowane wersje były podobne lub identyczne.
Aby losowo wygenerować klucz lub IV, należy użyć klasy `RandomNumberGenerator` w `System.Cryptography`.

```csharp
byte[] key = new byte[16];
byte[] iv = new byte[16];
RandomNumberGenerator rand = RandomNumberGenerator.Create();
rand.GetBytes(key);
rand.GetBytes(iv);

```

Jeśli nie podamy klucza ani IV, to zostaną wygenerowane losowe wartości o dużej sile szyfrowania. Można je przeglądać korzystając z właściwości Key i IV obiektu `Aes`.
#### Dekoratory szyfrujące

Zadanie szyfrowania jest podzielone pomiędzy klasy:

* `Aes` jest obiektem szyfrującym (enkryptorem). Stosuje algorytm szyfrowania wraz z jego funkcjami szyfrowania i odszyfrowywania.

  - Można zastąpić Aes innym algorytmem symetrycznym i nadal używać `CryptoStream`

* Obiekt typu `CryptoStream` pełni rolę dekoratora strumienia i może być łączony łańcuchowo z innymi obiektami dziedziczącymi po klasie `Stream`

  - `CryptoStream` jest dwukierunkowy – umożliwia odczyt strumienia i zapis do  strumienia, zależnie od wybranego trybu (CryptoStreamMode.Read lub CryptoStreamMode.Write)
Zarówno enkryptory, jak jak dekryptory mogą odczytywać i zapisywać, co daje cztery kombinacje
W razie wątpliwości najlepiej wybrać `Write` dla kodowania, a `Read` dla dekodowania

```csharp
using (Aes algorithm = Aes.Create())
{
    using (ICryptoTransform encryptor = algorithm.CreateEncryptor())
    using (Stream f = File.Create("serious.bin"))
    using (Stream c = new CryptoStream(f, encryptor,CryptoStreamMode.Write))
    using (Stream d = new DeflateStream(c, CompressionMode.Compress))
    using (StreamWriter w = new StreamWriter(d))
        w.WriteLine("Small and secure!");

    using (ICryptoTransform decryptor = algorithm.CreateDecryptor())
    using (Stream f = File.OpenRead("serious.bin"))
    using (Stream c = new CryptoStream(f, decryptor, CryptoStreamMode.Read))
    using (Stream d = new DeflateStream(c, CompressionMode.Decompress))
    using (StreamReader r = new StreamReader (d))
        Console.WriteLine(r.ReadLine()); // Krótkie i bezpieczne
}

```

## Szyfrowanie antysymetryczne

* Szyfrowanie kluczem publicznym jest asymetryczne
* Do szyfrowania jest używany inny klucz niż do odszyfrowywania
* W odróżnieniu od szyfrowania symetrycznego, gdzie dowolna seria bajtów o odpowiedniej długości może służyć za klucz, w szyfrowaniu symetrycznym wymagana jest specjalnie wytworzona para kluczy
* Para kluczy zawiera klucz prywatny i klucz publiczny, które działają w następujący sposobów:

  - Klucz publiczny szyfruje wiadomości
  - Klucz prywatny odszyfrowuje wiadomości

* Strona "wytwarzająca" parę kluczy nie ujawnia klucza prywatnego, a klucz publiczny dystrybuuje bez ograniczeń
* Brak możliwości obliczenia klucza prywatnego na podstawie klucza publicznego

  - Jeśli klucz prywatny zaginie, nie będzie można odtworzyć danych
  - W razie wycieku klucza prywatnego, system szyfrowania stanie się bezużyteczny 

* Wymiana klucza publicznego umożliwia bezpieczną komunikację pomiędzy dwoma komputerami po sieci publicznej

  - Nie wymaga wcześniejszego kontaktu ani dzielenia tajnych danych
### RSA

* Technika RSA (Rivest, Shamir i Adelman) została opracowana przez RSA Security LLC
* Używa kluczy w trzech rozmiarach:

  - 1024 bity
  - 2046 bity (minimum we współczesnych systemach szyfrowania)
  - 4096 bity
Przykład użycia klasy RSA:

```csharp
byte[] data = { 1, 2, 3, 4, 5 }; // To, co szyfrujemy

using (var rsa = new RSACryptoServiceProvider())
{
    byte[] encrypted = rsa.Encrypt(data, true);
    byte[] decrypted = rsa.Decrypt(encrypted, true);
}

```

Ponieważ nie określiliśmy ani publicznego, ani prywatnego klucza, więc dostawca szyfrowania wygenerował parę kluczy używając domyślnej długości 1024 bitów. Można wygenerować dłuższe klucze ze skokiem o 8 bajtów za pomocą konstruktora
Dla aplikacji krytycznych pod względem bezpieczeństwa zaleca się wygenerować klucz o długości 2048 bitów

```csharp
var rsa = new RSACryptoServiceProvider(2048);

```

Generowanie pary kluczy wymaga mocy obliczeniowej – zabiera  około 100 ms. Z tego względu implementacja RSA wstrzymuje generowanie kluczy do momentu, kiedy są na prawdę potrzebne, na przykład, do czasu wywołania Encrypt()
Daje to możliwość załadowania istniejącego klucza lub pary kluczy:

* Metody `ImportCspBlob()` i `ExportCspBlob()` ładują i zapisują klucze w formacie tablicy bajtów
* Metody `FromXmlString()` i `ToXmlString()` działają analogicznie dla ciągu znaków i dają w wyniku ciąg znaków zawierający znacznik XML
* Flaga `boolean` pozwala oznaczyć, czy klucz prywatny powinien zostać uwzględniony przy zapisie.

Wytwarzanie pary i kluczy i zapisywanie jej na dysku:

```csharp
using (var rsa = new RSACryptoServiceProvider())
{
    File.WriteAllText("PublicKeyOnly.xml", rsa.ToXmlString(false));
    File.WriteAllText("PublicPrivate.xml", rsa.ToXmlString(true));
}

```

Ponieważ nie podaliśmy istniejących kluczy, `ToXmlString()` wymusił wytworzenie nowej pary kluczy (przy pierwszym wywołaniu)

```csharp
byte[] data = Encoding.UTF8.GetBytes("Message to encrypt");

string publicKeyOnly = File.ReadAllText("PublicKeyOnly.xml");
string publicPrivate = File.ReadAllText("PublicPrivate.xml");

byte[] encrypted, decrypted;

using (var rsaPublicOnly = new RSACryptoServiceProvider())
{
    rsaPublicOnly.FromXmlString(publicKeyOnly);
    encrypted = rsaPublicOnly.Encrypt (data, true);
    /* Następna linia rzuci wyjątek, bo potrzebny jest klucz prywatny
       do odszyfrowania: decrypted = rsaPublicOnly.Decrypt (encrypted, true); */
}

using (var rsaPublicPrivate = new RSACryptoServiceProvider())
{
    // Mając klucz prywatny możemy odszyfrować:
    rsaPublicPrivate.FromXmlString(publicPrivate);
    decrypted = rsaPublicPrivate.Decrypt(encrypted, true);
}

```

### Szyfrowanie hybrydowe

Załóżmy, że komputer Origin chce przesłać poufną wiadomość do komputera Target:
1. Target generuje parę kluczy i przesyła swój klucz publiczny do Origin
2. Origin koduje poufną wiadomość za pomocą klucza publicznego Target i wysyła ją do Target
3. Target odkodowuje poufną wiadomość za pomocą swojego klucza prywatnego
Osoba postronna zobaczy klucz publiczny Target i tajną wiadomość zakodowaną kluczem publicznym Target, ale bez klucza prywatnego Target wiadomość nie może zostać odkodowana.
Tajna wiadomość przesłana z Origin do Target zwykle zawiera nowy klucz dla kolejnego szyfrowania symetrycznego
Pozwala to na użycie w dalszej części sesji szyfrowania symetrycznego, które jest wydajniejsze
Ten protokół  jest wyjątkowo bezpieczny, jeśli generowana jest nowa para kluczy dla każdej sesji i nie ma potrzeby przechowywania kluczy na komputerach.
## Podpisy cyfrowe

Algorytmy klucza publicznego mogą być również stosowane do cyfrowego podpisywania wiadomości lub dokumentów. Podpis jest podobny do tokenu klucza publicznego z tą różnicą, że jego wytworzenie wymaga klucza prywatnego i nie może zostać podrobiony. Klucz publiczny jest używany do zweryfikowania podpisu.
Podpisywanie polega na haszowaniu danych i zastosowaniu asymetrycznego algorytmu do otrzymanego skrótu (wartości hash).
Dzięki niewielkiemu rozmiarowi skrótu można szybko podpisywać duże dokumenty (szyfrowanie z użyciem klucza publicznego jest bardziej pamięciożerne). 

```csharp
byte[] data = Encoding.UTF8.GetBytes("Message to sign");
byte[] publicKey;
byte[] signature;
object hasher = SHA1.Create(); // wybrany algorytm hashowania

// Generuje nową parę kluczy, a następnie podpisuje dane za ich pomocą
using (var publicPrivate = new RSACryptoServiceProvider())
{
    signature = publicPrivate.SignData(data, hasher);
    publicKey = publicPrivate.ExportCspBlob(false); // generuje klucz publiczny
}

// Tworzy nowy obiekt RSA z użyciem klucza publicznego, potem weryfikuje podpis
using (var publicOnly = new RSACryptoServiceProvider())
{
    publicOnly.ImportCspBlob(publicKey);
    Console.Write (publicOnly.VerifyData(data, hasher, signature)); // true

    // Modyfikujemy dane i sprawdzamy podpis
    data[0] = 0;
    Console.Write(publicOnly.VerifyData(data, hasher, signature)); // false

    // Rzuca wyjątkiem ponieważ brakuje klucza prywatnego
    signature = publicOnly.SignData (data, hasher);
}

```

Można też samodzielnie dokonać haszowania danych, a następnie wywołać `SignHash` zamiast `SignData`:

```csharp
using (var rsa = new RSACryptoServiceProvider())
{
    byte[] hash = SHA1.Create().ComputeHash(data);
    signature = rsa.SignHash(hash, CryptoConfig.MapNameToOID("SHA1"));
    ...
}

```

* `SignHash` musi wiedzieć, który algorytm skrótu został zastosowany

  - `CryptoConfig.MapNameToOID()` dostarcza tej informacji w poprawnym formacie z nazwy takiej jak "SHA1"

* `RSACryptoServiceProvider` wytwarza podpisy o rozmiarach odpowiadających kluczowi
* Obecnie żaden z rozpowszechnionych algorytmów nie wytwarza bezpiecznych kluczy o rozmiarze znacznie mniejszym niższym 128 bajtów (odpowiednim do kodów aktywacyjnych produktów)
