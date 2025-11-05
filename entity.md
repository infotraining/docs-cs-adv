# Entity Framework
## Wprowadzenie

Platforma **Entity Framework (EF)** - to rozszerzenie platformy `.NET` umożliwiające zastosowanie w tworzonym oprogramowaniu odwzorowania obiektowo-relacyjnego (ang. `object-relational mapping`, w skrócie `O/RM` lub `ORM`).
Odwzorowanie obiektowo-relacyjne to sposób odwzorowania obiektowej architektury systemu informatycznego na bazę danych (lub inny element systemu) o relacyjnym charakterze.
![image](_images/oop-orm-db.*)

Podczas użytkowania relacyjnych baz danych z poziomu systemu informatycznego napisanego w obiektowym języku programowania pojawiają się rozmaite problemy zarówno konceptualne jak i techniczne. Określane są one mianem niedopasowania obiektowo-relacyjnego (ang. `object-relational impedance mismatch`).
### Platforma `Entity Framework`

* Umożliwia pracę z danymi przechowywanymi w bazie danych za pomocą tylko obiektów reprezentujących pojęcia domeny aplikacji. 
* Eliminuje potrzebę pisania dodatkowego kodu do obsługi operacji odczytu z/zapisu do bazy danych.
* Platforma `Entity Framework` jest narzędziem `OR/M` rozwiązującym problem niedopasowania obiektowo-relacyjnego.

![image](_images/ef/image15.*)

Zastosowanie platformy `Entity Framework`:

* Oddziela warstwę pojęć domeny aplikacji od warstwy dostępu do bazy danych
* Zwiększa produktywność programistów:
  * Przetwarzają jedynie obiekty (pojęcia domeny aplikacji)
  * Odczyt (zapis) danych z bazy danych do obiektów jest realizowany automatycznie (przez platformę `EF`)
* Daje możliwość łatwej zmiany zastosowanego źródła danych (np. `Microsoft SQL Server`, `Oracle`)

Platforma `Entity Framework`:

* Nie rozwiązuje wszystkich problemów
* Nie jest najszybszym sposobem dostępu do danych
* Nie jest jedyną platformą `O/RM` dla platformy `.NET`. Przykładowe inne platformy/narzędzia to: `DataObjects.Net`, `NHibernate`, `OpenAccess ORM`
* Jest strategiczną technologią Microsoftu: `WCF Data Services`, `Azure Table Storage`, `Sharepoint 2010`, `SQL Server Reporting Services and PowerPivot for Excel`, …

`ADO.NET` wciąż jest w użyciu:

* Obiekty typu `DbDataReader` i `DataSet` wciąż mogą być wykorzystywane w aplikacjach
* `ADO.NET` jest m.in. używany w platformie `EF`

Historia platformy `Entity Framework`:

* Typed Dataset
* LINQ to SQL (.NET 3.5 – VS 2008)
* LINQ to Entities  i EF (=EF 3.5) (NET 3.5 SP1 – VS 2008 SP1)
* EF 4 (.NET 4.0 – VS 2010), 4.1, 4.1.1, 4.2, 4.3, 4.3.1
* EF 5 (.NET 4.x – VS 2010/12)
* EF 6, 6.0.1, 6.0.2, 6.1.0 (.NET 4.x – VS 2010/12/13)

Oficjalna strona platformy `Entity Framework`: `http://msdn.microsoft.com/data/ef/ <http://msdn.microsoft.com/data/ef/>`_
Blog zespołu `ADO.NET`: `http://blogs.msdn.com/b/adonet/ <http://blogs.msdn.com/b/adonet/>`_
Strona projektu `EF` w serwisie `CodePlex`: `http://entityframework.codeplex.com/ <http://entityframework.codeplex.com/>`
## Entity Data Model

Architektura platformy `Entity Framework`:
![image](_images/ef/image17.*)

Dostawca danych `EntityClient` (ang. `EntityClient data provider`) - jest to dostawca danych używany przez aplikacje platformy `.NET`, które za pomocą platformy `Entity Framework` uzyskują dostęp do danych opisanych w konceptualnym modelu `EDM` (ang. `Entity Data Model`).
Model **Entity Data Model** jest wzorowany na modelu związków encji `ERM` (ang. `entity–relationship model`), opublikowanym przez Petera Chen w 1976 roku, a opisującym bazę danych w abstrakcyjny sposób za pomocą encji i związków (relacji) występujących pomiędzy nimi. Często zamiast pojęcia modelu związków encji `ERM` używa się pojęcia diagram związków encji `ERD` (ang. `entity-relationship diagram`) reprezentującego graficznie model `ERM`.
Model `Entity Data Model`:

* Model danych z punktu widzenia klienta bazy danych (np. aplikacji).
* Opisuje strukturę obiektów domeny aplikacji (tzw. obiektów biznesowych), które też są tu nazywane encjami.
* Nie jest modelem związków encji `ERM`, bo nie opisuje modelu bazy danych, chociaż zawiera model logiczny bazy danych.

Model `ERM` w bazie danych:
![image](_images/ef/image18.*)

Model `EDM` (model konceptualny) w aplikacji:
![image](_images/ef/image19.*)

Model `EDM` składa się z 3 warstw:

* Warstwa modelu konceptualnego (ang. `Conceptual Layer)` - abstrakcyjna (niezależna od bazy danych) specyfikacja typów encji (klas), asocjacji (referencji) i innych elementów występujących w domenie aplikacji. Model konceptualny jest opisany za pomocą języka `CSDL`. Umożliwia zdefiniowanie relacji jeden-do-jednego, jeden-do-wielu, wiele-do-wielu, dziedziczenia, typów złożonych, typów wyliczeniowych itd.
* Warstwa odwzorowań (ang. `Mapping Layer`) - zawiera odwzorowania elementów modelu konceptualnego w model logiczny. Jest opisana za pomocą języka `MSL`
* Warstwa modelu logicznego (ang. `Storage Layer`) - model struktury bazy danych składający się z tabel, widoków, zdalnych procedur oraz ich wzajemnych zależności i kluczy. Jest opisany za pomocą języka `SSDL`

Zależności między warstwami modelu `EDM`:
![image](_images/ef/image20.*)

Definiowany w pliku `XML` (`.edmx`):
![image](_images/ef/image21.*)

## Modelowanie encji i asocjacji

Sposoby modelowania encji i asocjacji w `EDM`:

* Jawnie w pliku `.edmx` za pomocą narzędzia do projektowania modeli `EDM` (ang. `EF Designer`)
* Niejawnie przez platformę `EF`, która utworzy dynamicznie model `EDM` na podstawie kodu istniejących klas:

  - platforma `EF` tworzy model `EDM` zgodnie z określoną konwencją, której zasady można przesłaniać

* W wersji `EF6` można tworzyć własne konwencje

Scenariusze tworzenia modelu `EDM`:

* Jawne tworzenie modelu `EMD`

  - `Model First` (tworzenie nowej bazy danych)
  - `Database First` (na podstawie istniejącej bazy danych)

* Niejawne tworzenie modelu `EMD`

  - `Code First` (dwie wersje scenariusza: tworzenie nowej bazy danych lub na podstawie istniejącej bazy danych)
Model `EMD`:
![image](_images/ef/image22.*)

Skrypt `DDL SQL` tworzący bazę danych:
![image](_images/ef/image23.*)

Istniejąca baza danych:
![image](_images/ef/image24.*)

Klasy encji (tzw. klasy `POCO` = `Plain Old CLR Objects`):
![image](_images/ef/image25.*)

### Sceneriusz Model First

Model konceptualny tworzony jest jawnie przez programistę za pomocą narzędzia do projektowania modeli `EDM`. Na jego podstawie generowany jest skrypt `DDL SQL` tworzący bazę danych oraz generowane są klasy odpowiadające encjom.
![image](_images/ef/image22232425.*)

**Krok 1**: Dodanie do projektu nowego elementu projektu `ADO.NET Entity Data Model`:
![image](_images/ef/image30.*)

**Krok 2**: Podczas dodawania pliku `.edmx` należy wybrać opcję wygenerowania pustego modelu:
![image](_images/ef/image31.*)

**Krok 3**: Po zbudowaniu modelu `EDM` zapis pliku automatycznie inicjuje generowanie klas `POCO`:
![image](_images/ef/image32.*)

Krok 4: Aby wygenerować skrypt `DDL SQL` generujący bazę danych należy wybrać odpowiednią pozycję z menu kontekstowego:
![image](_images/ef/image33.*)

### Scenariusz Database First

Model konceptualny tworzony jest jawnie za pomocą narzędzia do projektowania modeli `EDM` stosując inżynierię wsteczną względem logicznego modelu bazy danych. Na jego podstawie generowane są klasy odpowiadające encjom.
![image](_images/ef/image222425.*)

**Krok 1**: Dodanie do projektu nowego elementu projektu `ADO.NET` `Entity Data Model`:
![image](_images/ef/image30.*)

**Krok 2**: Podczas dodawania pliku `.edmx` należy wybrać opcję wygenerowania modelu na podstawie bazy danych:
![image](_images/ef/image34.*)

**Krok 3**: Po określeniu połączenia do bazy danych należy wybrać, które obiekty bazodanowe powinny zostać przeniesione do modelu:
![image](_images/ef/image35.*)

**Krok 4**: Po zbudowaniu modelu `EDM` zapis pliku automatycznie inicjuje generowanie klas `POCO`:
![image](_images/ef/image32.*)

### Scenariusz Code First

Model konceptualny tworzony jest niejawnie przez platformę `EF`, która tworzy model `EDM` na podstawie istniejących klas encji.
Platforma `EF` (domyślnie) utworzy bazę danych w trakcie działania aplikacji, gdy jej nie ma.
![image](_images/ef/image222425_2.*)

**Krok 1**: Zainstalowanie platformy `EF` w projekcie za pomocą okna `Package Manager Console`:
```sh
Install-Package EntityFramework -Version 6.1.0 -Project CodeFirst

```

**Krok 2**: Utworzenie kodu klas encji (`POCO`):

```csharp
[Table("Produkty")]
public class Produkt
{
    public int Id { get; set; }
    public string Nazwa { get; set; }
    public decimal Cena { get; set; }
    public string Opis { get; set; }
    public int KategoriaId { get; set; }
    public virtual Kategoria Kategoria { get; set; }
}

```

```csharp
[Table("Kategorie")]
public class Kategoria
{
    public int Id { get; set; }
    public string Nazwa { get; set; }
    public string Opis { get; set; }
    public virtual ICollection<Produkt> Produkty { get; set; }

    public Kategoria() { Produkty = new HashSet<Produkt>(); }
}

```

**Krok 3**: Utworzenie kodu klasy kontekstu:

```csharp
public class Sklep : DbContext
{
    public Sklep() : base("szkolenie") { }

    public DbSet<Kategoria> Kategorie { get; set; }
    public DbSet<Produkt> Produkty { get; set; }
}

```

Istnieje możliwość wygenerowania klas na podstawie istniejącej bazy danych dzięki narzędziu `Entity Framework Power Tools Beta 4`.

* Należy w oknie `SolutionExplorer` zaznaczyć projekt i z menu kontekstowego wybrać opcję `Entity Framework`, a następnie `Reverse Engineer Code First`.

W przypadku, gdy klasy `POCO` nie stosują się do domyślnej konwencji platformy `EF`, konieczne jest dokonfigurowanie generowanego dynamicznie modelu `EDM`. Dostępne są dwie techniki:

* `Data Annotations` – adnotacje dla klas za pomocą atrybutów
* `Fluent API` – interfejs umożliwiający sterowanie tzw. "budowniczym modelu" `EDM`
### Data Annotations

* Technika konfigurowania klasami `POCO` za pomocą atrybutów – adnotacji
* Proste rozwiązanie o ograniczonych możliwościach
* Rozwiązanie łamiące zasadę określającą, że klasy `POCO` nic "nie wiedzą" o sposobie ich zapisu do bazy danych (ang. `Persistance Ignorance`)

Wybrane atrybuty:
`Table` – określa bazodanową nazwę tabeli, w której zapisywane są dane obiektu konkretnej klasy

```csharp
[Table("Kateg")]
public class Kategoria
{
    (...)
}

```

`Column` – określa bazodanową nazwę kolumny, w której zapisywane są wartości danej właściwości

```csharp
[Column("Info", TypeName = "ntext")]
public string Opis { get; set; }

```

`Key` – kolumna odpowiadająca danej właściwości stanowi klucz w bazodanowej tabeli

```csharp
[Key]
public int Nr { get; set; }

```

`Required` – właściwość wymaga podania wartości

```csharp
[Required]
public string Nazwa { get; set; }

```

`Max/MinLength` – określa maksymalną/minimalną długość właściwości typu string

```csharp
[MaxLength(10), MinLength(5)]
public string Kod { get; set; }

```

### Fluent API

Interfejs umożliwiający określenie konfiguracji modelu `EDM` bezpośrednio w klasie odpowiedzialnej za tworzenie tego modelu. Daje większe możliwości niż `Data Annotations`. Umożliwia oddzielenie klas `POCO` od ustawień konfiguracji tych klas pod kątem ich bazodanowego zapisu/odczytu.
W klasie kontekstu należy nadpisać metodę `OnModelCreating` i następnie rozpocząć konfigurację za pomocą obiektu klasy `DbModelBuilder`.

```csharp
public class Sklep : DbContext
{
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        (...)
    }
}

```

Przykłady:
Określenie nazwy bazodanowej tabeli

```csharp
modelBuilder.Entity<Kategoria>().("Kateg");

```

Określenie nazwy bazodanowej kolumny

```csharp
modelBuilder.Entity<Kategoria>().Property(k => k.Opis).HasColumnName("Info");

```

Określenie typu bazodanowej kolumny

```csharp
modelBuilder.Entity<Kategoria>().Property(k => k.Opis).HasColumnType("ntext");

```

Określenie kolumny odpowiadająca danej właściwości jako klucz w bazodanowej tabeli

```csharp
modelBuilder.Entity<Kategoria>().HasKey(k => k.Nr);

```

Określenie, że właściwość wymaga podania wartości

```csharp
modelBuilder.Entity<Kategoria>().Property(k => k.Nazwa).IsRequired();

```

Określenia maksymalnej długości właściwości typu `string`

```csharp
modelBuilder.Entity<Kategoria>().Property(k => k.Kod).HasMaxLength(10);

```

### Modelowanie za pomocą narzędzia do projektowania modeli EDM

Dodanie elementów modelu, np. encji:
![image](_images/ef/image383940.*)

Dodanie nowych właściwości do encji:
![image](_images/ef/image41.*)

Dodanie do modelu asocjacji/dziedziczenia:
![image](_images/ef/image384243.*)

Dodanie do modelu typów złożonych, typów wyliczeniowych:
![image](_images/ef/image44.*)

Modyfikowanie wszystkich elementów modelu za pomocą pomocniczego okna `Model Browser`:
![image](_images/ef/image45.*)

Modyfikowanie odwzorowań dla wybranej encji za pomocą pomocniczego okna `Mapping Details`:
![image](_images/ef/image46.*)

Na podstawie modelu `EDM` można wygenerować kod klas `POCO`. Jest to realizowane przez narzędzie `Entity Model Code Generator`. Wykorzystuje ono silnik `T4` (ang. `Text Template Transformation Toolkit`) – generator wbudowany w `Visual Studio`. Kod generowany jest na podstawie szablonu zapisanego w pliku o rozszerzeniu `.tt`.
Generowanie kodu domyślnie jest włączone przy każdym zapisie pliku modelu `EDM`. Można zmienić to zachowanie we właściwościach modelu `EDM` (ustawienie `Transform Related Text Templates On Save`). Obecnie `Visual Studio` (2012 oraz 2013) domyślnie dołącza do modelu `EDM` szablon `EF x.x DbContext Generator`.
![image](_images/ef/image47.*)

Można zmienić sposób generowania klas modyfikując przypisane szablony (pliki `.tt`).
![image](_images/ef/image48.*)

Można też skorzystać z innych szablonów. Należy w tym celu dodać do modelu `EDM` nowy szablon (ang. `Code Generation Item`).
![image](_images/ef/image4950.*)

## Odwzorowania O/RM
### Modyfikacje encji

![image](_images/ef/modyfying-entities.*)

### Podział encji (ang. `entity splitting`)

![image](_images/ef/entity-splitting.*)

Jednej encji w modelu `EDM` odpowiadają dwie lub więcej tabele w bazie danych:
![image](_images/ef/entity-model-edm.*)

Należy zadbać o odpowiednie powiązania w oknie `Mapping Details`:
![image](_images/ef/mapping-details.*)

### Podział tabeli (ang. `table splitting`)

![image](_images/ef/table-splitting.*)

Jednej tabeli w bazie danych odpowiadają dwie lub więcej encji w modelu `EDM`:
![image](_images/ef/table-model-edm.*)

Należy zadbać o odpowiednie powiązania w oknie `Mapping Details` oraz ustawienie referencji we właściwościach asocjacji:
![image](_images/ef/mapping-details-associations.*)

### Typy złożone (ang. `complex types`)

![image](_images/ef/complex-types.*)

Typy złożone widoczne są tylko w oknie `Model Browser`:
![image](_images/ef/complex-types-model-browser.*)

Należy zadbać o odpowiednie powiązania w oknie `Mapping Details`:
![image](_images/ef/mapping-details-associations-2.*)

### Odwzorowanie warunkowe

![image](_images/ef/filter-condition.*)

Warunek filtrujący definiuje się w oknie `Mapping Details`:
![image](_images/ef/filter-condition-mapping-details.*)

Asocjacje zwrotne:
![image](_images/ef/recursive-association.*)

### Relacja dziedziczenia między encjami-obiektami

![image](_images/ef/entities-objects-inheritance.*)

Wyróżnia się 3 sposoby realizacji odwzorowania relacji dziedziczenia między encjami-obiektami w relacyjnej bazie danych:

* TPT (ang. `table-per-type`)
* TPH (ang. `table-per-hierarchy`)
* TPC (ang. `table-per-concrete type`)

Można stosować też rozwiązanie mieszane. Możliwe jest to tylko w podejściu `Code First`.
**Dziedziczenie TPT**
Dostępne jest w każdym podejściu (`Model First`, `Database First`, `Code First`). Każdej encji przypisywana jest osobna tabela w bazie danych zawierająca tylko wartości nieodziedziczonych właściwości.
![image](_images/ef/tpt-inheritance.*)

Zalety dziedziczenia `TPT` to:

* Elastyczność (możliwość modyfikacji wybranej encji/tabeli bez naruszania struktur pozostałych)
* Bazodanowa walidacja (możliwość bazodanowej kontroli wartości w każdej tabeli)
* Efektywne wykorzystanie przestrzeni dyskowej (brak nadmiarowych pustych pól)
* Estetyka (struktura bazy danych przypomina hierarchię encji w aplikacji)

Wadą jest niska wydajność zapytań (szybko pogarszająca się wraz ze wzrostem hierarchii dziedziczenia). Oto przykład zapytania odczytującego wszystkie samochody:
```sql
SELECT
CASE WHEN ([UnionAll1].[C2] = 1) THEN '0X0X' ELSE '0X1X' END AS [C1],
[UnionAll1].[Id] AS [C2],
[Extent3].[Marka] AS [Marka],
[Extent3].[RokProdukcji] AS [RokProdukcji],
CASE WHEN ([UnionAll1].[C2] = 1) THEN [UnionAll1].[LiczbaMiejsc] END AS [C3],
CASE WHEN ([UnionAll1].[C2] = 1) THEN CAST(NULL AS float) ELSE [UnionAll1].[C1] END AS [C4]
FROM (SELECT
    [Extent1].[Id] AS [Id],
    [Extent1].[LiczbaMiejsc] AS [LiczbaMiejsc],
    CAST(NULL AS float) AS [C1],
    cast(1 as bit) AS [C2]
    FROM [dbo].[Samochody_TptOsobowy] AS [Extent1]
UNION ALL
    SELECT
    [Extent2].[Id] AS [Id],
    CAST(NULL AS smallint) AS [C1],
    [Extent2].[Ladownosc] AS [Ladownosc],
    cast(0 as bit) AS [C2]
    FROM [dbo].[Samochody_TptCiezarowy] AS [Extent2]) AS [UnionAll1]
INNER JOIN [dbo].[Samochody] AS [Extent3] ON [UnionAll1].[Id] = [Extent3].[Id]

```

Oto przykład zapytania odczytującego tylko samochody osobowe:
```sq;
SELECT
'0X0X' AS [C1],
[Extent1].[Id] AS [Id],
[Extent2].[Marka] AS [Marka],
[Extent2].[RokProdukcji] AS [RokProdukcji],
[Extent1].[LiczbaMiejsc] AS [LiczbaMiejsc]
FROM [dbo].[Samochody_TptOsobowy] AS [Extent1]
INNER JOIN [dbo].[Samochody] AS [Extent2] ON [Extent1].[Id] = [Extent2].[Id]  

```

**Dziedziczenie TPH**
Dostępne standardowo tylko w podejściu `Database First` oraz `Code First`. W podejściu `Model First` konieczne jest przygotowanie nowego szablonu (ang. `database generation workflow`) dla automatu generującego skrypt `DDL SQL` tworzący bazę danych.
![image](_images/ef/tph-inheritance.*)

Tworzona jest jedna wspólna tabela dla wszystkich encji zawierająca kolumny wszystkich encji. Jedna dodatkowa kolumna (domyślnie nazwana `Discriminator`) określa typ encji dla danego wiersza. Na podstawie tej kolumny określa się warunki filtrujące dla encji pochodnych po encji bazowej.
![image](_images/ef/tph-inheritance-2.*)

Jest to zalecany sposób realizacji dziedziczenia encji (zwłaszcza w małych projektach), pomimo braku wielu zalet `TPT`. Argumentem przemawiającym za tym zaleceniem jest uzyskiwanie najlepszej wydajności zapytań w porównaniu do pozostałych sposobów (brak unii i złączeń w zapytaniach).
Oto przykład zapytania odczytującego wszystkie samochody:
```sql
SELECT
[Extent1].[Discriminator] AS [Discriminator],
[Extent1].[Id] AS [Id],
[Extent1].[Marka] AS [Marka],
[Extent1].[RokProdukcji] AS [RokProdukcji],
[Extent1].[LiczbaMiejsc] AS [LiczbaMiejsc],
[Extent1].[Ladownosc] AS [Ladownosc]
FROM [dbo].[TphSamochody] AS [Extent1]
WHERE [Extent1].[Discriminator] IN (N'TphOsobowy',N'TphCiezarowy')

```

Oto przykład zapytania odczytującego tylko samochody osobowe:
```sql
SELECT
'0X0X' AS [C1],
[Extent1].[Id] AS [Id],
[Extent1].[Marka] AS [Marka],
[Extent1].[RokProdukcji] AS [RokProdukcji],
[Extent1].[LiczbaMiejsc] AS [LiczbaMiejsc]
FROM [dbo].[TphSamochody] AS [Extent1]
WHERE [Extent1].[Discriminator] = N'TphOsobowy'

```

**Dziedziczenie TPC**
Dostępne jest tylko w podejściu `Code First` (nie jest obsługiwane przez narzędzie do projektowania modeli `EDM`). Jest to rozwiązanie będące kompromisem pomiędzy sposobami `TPT` i `TPH`. Pojawiają się jednak inne problemy. Ponieważ wiele tabel ma wspólny identyfikator, istnieje problem zachowania jego unikalności i np. użycia wspólnego dla hierarchii autonumerowania. Każdej nieabstrakcyjnej encji przypisywana jest osobna tabela w bazie danych zawierająca wartości wszystkich (nawet tych odziedziczonych) właściwości.
![image](_images/ef/tpc-inheritance.*)

Zapytania nie są tak skomplikowane jak w `TPT` (jest unia wyników, które są jednak bez złączeń).
Oto przykład zapytania odczytującego wszystkie samochody:
```sql
SELECT
    CASE WHEN ([UnionAll1].[C2] = 1) THEN '0X0X' ELSE '0X1X' END AS [C1],
    [UnionAll1].[Id] AS [C2],
    [UnionAll1].[Marka] AS [C3],
    [UnionAll1].[RokProdukcji] AS [C4],
    CASE WHEN ([UnionAll1].[C2] = 1) THEN [UnionAll1].[C1] END AS [C5],
    CASE WHEN ([UnionAll1].[C2] = 1) THEN CAST(NULL AS smallint) ELSE
[UnionAll1].[LiczbaMiejsc] END AS [C6]
    FROM (SELECT
        [Extent1].[Id] AS [Id],
        [Extent1].[Marka] AS [Marka],
        [Extent1].[RokProdukcji] AS [RokProdukcji],
        CAST(NULL AS float) AS [C1],
        [Extent1].[LiczbaMiejsc] AS [LiczbaMiejsc],
        cast(0 as bit) AS [C2]
        FROM [dbo].[TpcOsobowe] AS [Extent1]
    UNION ALL
        SELECT
        [Extent2].[Id] AS [Id],
        [Extent2].[Marka] AS [Marka],
        [Extent2].[RokProdukcji] AS [RokProdukcji],
        [Extent2].[Ladownosc] AS [Ladownosc],
        CAST(NULL AS smallint) AS [C1],
        cast(1 as bit) AS [C2]
        FROM [dbo].[TpcCiezarowe] AS [Extent2]) AS [UnionAll1]

```

Oto przykład zapytania odczytującego tylko samochody osobowe:
```sql
SELECT
'0X0X' AS [C1],
[Extent1].[Id] AS [Id],
[Extent1].[Marka] AS [Marka],
[Extent1].[RokProdukcji] AS [RokProdukcji],
[Extent1].[LiczbaMiejsc] AS [LiczbaMiejsc]
FROM [dbo].[TpcOsobowe] AS [Extent1]

```

## Przetwarzanie danych

Podstawowym obiektem umożliwiającym przetwarzanie danych za pośrednictwem platformy `EF` jest tzw. kontekst. Implementuje on wzorce: repozytorium (ang. `repository`) i jednostkę pracy (ang. `unit of work`). Udostępnia kolekcje obiektów-encji na podstawie danych odczytanych z bazy danych. "Śledzi" zmiany w powiązanych z kolekcjami obiektach-encjach.
Zazwyczaj tworzy się własną klasę dziedziczącą po klasie `DbContext`, w której udostępnia się kolekcje encji-obiektów.
```c#
public class Sklep : DbContext
{
    public DbSet<KategoriaProduktu> KategorieProduktow { get; set; }
    public DbSet<Produkt> Produkty { get; set; }
    public DbSet<SiecSklepow> SieciSklepow { get; set; }
    public DbSet<Sklep> Sklepy { get; set; }
    public DbSet<LogoSieci> LogaSieci { get; set; }
    (...)
}

```

Kontekst zarządza połączeniami do bazy danych i dlatego należy dbać o zwalnianie jego zasobów po zakończeniu jego użytkowania.
```c#
using (MojeSklepy mojeSklepy = new MojeSklepy())
{
    (...)
}

```

Zaleca się tworzyć osobny kontekst działający przez cały czas życia każdej formatki `WinForms`/`WPF`, dzięki czemu kontekst bezproblemowo będzie "śledził" wprowadzane zmiany. W przypadku aplikacji WWW należy używać osobnego kontekstu dla każdego żądania (ang. `web request`). Ważne jest, aby wiedzieć, że kontekst nie śledzi zmian w "nieswoich encjach".
Odczyt danych ze źródła danych za pomocą platformy `EF` realizować można za pomocą tzw. `LINQ to Entities`.
![image](_images/ef/data-processing.*)

Kolekcje encji-obiektów - umożliwiają sekwencyjny odczyt wszystkich encji. Każdorazowy dostęp do kolekcji encji powoduje wykonanie osobnego zapytania bazodanowego.
```c#
foreach (var produkt in mojeSklepy.Produkty)
{ (...) }
foreach (var produkt in mojeSklepy.Produkty)
{ (...) }

```

Kolekcje encji-obiektów implementują m.in. interfejs `IEnumerable` dzięki czemu możliwe jest użycie `LINQ` (w przypadku platformy `EF` to jest `LINQ to Entities`):
```c#
var produktyNaC = from p in mojeSklepy.Produkty
                  where p.Nazwa.StartsWith("C")
                  select p;

foreach (var produkt in produktyNaC)
{ (...) }

```

Można stosować zarówno składnię zapytań jak i operatory zapytań połączone z wyrażeniami lambda:
```c#
//Składnia zapytań
var produktyNaC = from p in mojeSklepy.Produkty
                  where p.Nazwa.StartsWith("C")
                  select p;

//Operatory zapytań + wyrażenia lambda
produktyNaC = mojeSklepy.Produkty.Where(p => p.Nazwa.StartsWith("C"));

```

Kolekcje encji-obiektów implementują też interfejs `IQueryable`. Umożliwia to zdalne przetworzenie zapytania `LINQ`:
```c#
var produktyNaC = from p in mojeSklepy.Produkty
                  where p.Nazwa.StartsWith("C")
                  select p;

// Zliczanie wykonane zdalnie - zwrócony tylko wynik zliczania
int liczbaProduktowNaC = produktyNaC.Count();

// Zliczanie wykonane lokalnie po pobraniu wszystkich elementów spełniających zapytanie
liczbaProduktowNaC = produktyNaC.AsEnumerable().Count();

```

W `LINQ to Entities` można korzystać również z właściwości nawigujących:
```c#
var produktyKatF = from p in mojeSklepy.Produkty
                   where p.KategoriaProduktu.Kod.Grupa.StartsWith("F")
                   select p;

// Produkty kategorii, których kod-grupa zaczyna się na literę F
foreach (var produkt in produktyKatF)
{ (...) }

```

Encje wskazywane przez wirtualne właściwości nawigujące są ładowane na żądanie – jest to tzw. leniwe ładowanie (ang. `lazy loading`).
```c#
var produktyKatF = from p in mojeSklepy.Produkty
                   where p.KategoriaProduktu.Kod.Grupa.StartsWith("F")
                   select p;

// Produkty i kategorie, których kod-grupa zaczyna się na literę F
foreach (var produkt in produktyKatF)
{
    Console.WriteLine("{0} - {1}", produkt.Nazwa, produkt.KategoriaProduktu.Nazwa);
}

```

Można wymusić wcześniejsze załadowanie wszystkich encji wskazywanych przez wybrane właściwości nawigujące (metoda `Include`):
```c#
var produktyKatF = from p in mojeSklepy.Produkty.Include(p => p.KategoriaProduktu)
                   where p.KategoriaProduktu.Kod.Grupa.StartsWith("F")
                   select p;

// Produkty i kategorie, których kod-grupa zaczyna się na literę F
foreach (var produkt in produktyKatF)
{
    Console.WriteLine("{0} - {1}",
                      produkt.Nazwa, produkt.KategoriaProduktu.Nazwa);
}

```

"Leniwe ładowanie" można wyłączyć globalnie w konfiguracji kontekstu. Wtedy właściwości nawigujące nie wskazują na żadną encję (mają pustą wartość):
```c#
// Wyłączenie leniwego ładowania
mojeSklepy.Configuration.LazyLoadingEnabled = false;
foreach (var produkt in produktyKatF)
{
    Console.WriteLine("{0} - {1}", produkt.Nazwa, 
                      produkt.KategoriaProduktu.Nazwa);         //NullReferenceException
}

```

Aby wyłączyć dla danej właściwości nawigującej ładowanie na żądanie (leniwe ładowanie) należy nie deklarować jej jako wirtualną:
```c#
public partial class Produkt
{
    (...)
    public KategoriaProduktu KategoriaProduktu { get; set; }
    public ICollection<Sklep> Sklepy { get; set; }
}

```

Wtedy konieczne będzie ręczne załadowanie encji konkretnego typu do kontekstu. Można to wymusić w dowolnej chwili metodą `Load`:
```c#
// Wyłączenie leniwego ładowania
mojeSklepy.Configuration.LazyLoadingEnabled = false;

// Załadowanie wszystkich kategorii produktów do kontekstu
mojeSklepy.KategorieProduktow.Load();

foreach (var produkt in produktyKatF)
{
    Console.WriteLine("{0} - {1}", produkt.Nazwa, produkt.KategoriaProduktu.Nazwa);
}

```

Można też wymusić załadowanie w danej chwili encji wskazywanej przez konkretną właściwość nawigacyjną. Należy tu jednak uważać, bo są tu możliwe wielokrotne ładowania tej samej encji:
```c#
// Wyłączenie leniwego ładowania
mojeSklepy.Configuration.LazyLoadingEnabled = false;
foreach (var produkt in produktyKatF)
{
    // Załadowanie encji wskazywanej przez konkretną właściwość nawigacyjną
    mojeSklepy.Entry(produkt).Reference(p => p.KategoriaProduktu).Load();
    Console.WriteLine("{0} - {1}", produkt.Nazwa, produkt.KategoriaProduktu.Nazwa);
}

```

W przypadku encji będących w relacji dziedziczenia kolekcja encji bazowych zwraca również wszystkie encje pochodne. Aby odczytać obiekty tylko pochodnej encji należy użyć metody `OfType`.
```c#
foreach (var przecenionyProdukt in mojeSklepy.Produkty.OfType<PrzecenionyProdukt>())
{
    Console.WriteLine(przecenionyProdukt.Nazwa);
}

```

Każdorazowy dostęp do kolekcji encji powoduje wykonanie osobnego zapytania bazodanowego. Aby temu zapobiec można użyć właściwość `Local`, która nakaże korzystanie z lokalnej kopii danych.
```c#
foreach (var produkt in mojeSklepy.Produkty) 
{ (...) }
foreach (var produkt in mojeSklepy.Produkty.Local)
{ (...) }

```

### Dodawanie, modyfikowanie i usuwanie obiektów encji

Jak już wspomniano, `LINQ to Entities` umożliwia jedynie odczyt danych ze źródła danych. Dodawanie nowych encji, modyfikacja i kasowanie istniejących wymagają skorzystania z dodatkowych metod platformy `EF`. Sposób realizacji tych zmian jest zgodny z wzorcami repozytorium i jednostka pracy.
Są dwa sposoby tworzenia nowego obiektu encji:

* Jawne wywołanie konstruktora klasy encji – utworzony obiekt nie zapewnia automatycznie "leniwego ładowania" i "śledzenia zmian"
* Użycie metody `Create` kolekcji encji – o ile spełnione są wymagane warunki, tworzony jest obiekt klasy pochodnej, tzw. pośrednik (ang. `proxy`), który automatycznie zapewnia "leniwe ładowanie" i/lub "śledzenie zmian" właściwości nawigacyjnych itd.

Nowoutworzony obiekt encji należy następnie dodać do kolekcji encji. Po dodaniu encji do kolekcji encji może ona zacząć "leniwie" doładowywać powiązane ze sobą encje nawet, jeśli jeszcze nie wykonano fizycznego zapisu do bazy danych. Na końcu utworzone encje należy zapisać do bazy danych. Należy wywołać metodę kontekstu `SaveChanges`. W przypadku stosowania autonumeracji nastąpi wtedy uaktualnienie identyfikatorów. Oto dwa fragmenty kodu prezentujące zapis nowoutworzonej (oboma sposobami) encji do bazy danych.
```c#
// Utworzenie obiektu encji Produkt
Produkt produkt = new Produkt()
{
    Nazwa = "Trabant",
    Opis = "Stary Trabant",
    KategoriaId = kategoriaId
};

// Dodanie produktu do kolekcji encji
mojeSklepy.Produkty.Add(produkt);

// Zapis zmian do bazy danych
mojeSklepy.SaveChanges();

```

```c#
// Utworzenie obiektu pośrednika
Produkt produktProxy = mojeSklepy.Produkty.Create();

produktProxy.Nazwa = "Trabancik";
produktProxy.Opis = "Stary Trabancik";
produktProxy.KategoriaId = kategoriaId;

// Dodanie produktu do kolekcji encji
mojeSklepy.Produkty.Add(produktProxy);

// Zapis zmian do bazy danych
mojeSklepy.SaveChanges();

```

Modyfikację istniejącego obiektu encji realizuje się w ramach kontekstu, do którego przynależy. Jeśli jest potrzeba zapisania zmodyfikowanej encji po zwolnieniu kontekstu, który ją utworzył, należy:

* Utworzyć nowy kontekst i uzyskać w tym kontekście nową encję o tym samym identyfikatorze.
* Następnie należy przepisać zmienione wartości do nowej encji i zapisać nową encję w ramach jej istniejącego kontekstu w bazie danych.

```c#
using (MojeSklepy mojeSklepy = new MojeSklepy())
{
    var produkt = mojeSklepy.Produkty.FirstOrDefault(p => p.Nazwa.StartsWith("Traban"));

    if (produkt != null)
    {
        // znaleziono produkt do zmodyfikowania
        produkt.Nazwa = "Nowa nazwa";
        produkt.Opis = "Nowy opis";

        // zapis zmodyfikowanej encji do bazy danych
        mojeSklepy.SaveChanges();
    }
}

```

A oto fragment kodu prezentujący modyfikację encji w ramach obcego kontekstu:
```c#
Produkt produkt;
using (MojeSklepy mojeSklepy = new MojeSklepy())
{
    produkt = mojeSklepy.Produkty.FirstOrDefault(p => p.Nazwa.StartsWith("Traban"));
}
if (produkt != null)
{ 
    // Modyfikacja poza kontekstem
    produkt.Nazwa = "Zmodyfikowana nazwa";
    produkt.Opis = "Zmodyfikowany opis";

    using (MojeSklepy mojeSklepy = new MojeSklepy())
    {
        var aktualnaKopia = mojeSklepy.Produkty.Find(produkt.Id);

        if (aktualnaKopia != null)
        {
            // Zapis zmodyfikowanej encji do bazy danych
            aktualnaKopia.Nazwa = produkt.Nazwa;
            aktualnaKopia.Opis = produkt.Opis;
            mojeSklepy.SaveChanges();
        }
    }
}

```

Podczas modyfikacji poza kontekstem zazwyczaj korzysta się z obiektów `DTO` (ang. `data transfer object`). Umożliwiają one serializację encji (nawet jeśli jest dynamicznie wygenerowanych obiekt pośrednika będący obiektem klasy pochodnej), co jest istotne w przypadku użycia platformy `EF` w aplikacjach wielowarstwowych. Do wyszukiwania istniejącej encji po kluczu (unikalnym identyfikatorze) służy metoda `Find`. Ważne jest, że dla nowoutworzonych encji stosuje się identyfikator równy 0.
Usuwanie istniejącego obiektu encji realizuje się w ramach kontekstu, do którego przynależy:
```c#
using (MojeSklepy mojeSklepy = new MojeSklepy())
{
    var produkt = mojeSklepy.Produkty.FirstOrDefault(p => p.Nazwa.StartsWith("Traban"));

    if (produkt != null)
    {
        // znaleziono produkt do usunięcia
        mojeSklepy.Produkty.Remove(produkt);

        // Realizacja operacji usunięcia w bazie danych
        mojeSklepy.SaveChanges();
    }
}

```

### Mechanizm śledzenia encji

Mechanizm śledzenia encji (ang. `change tracking`) pozwala rozpoznać przeprowadzone w kontekście zmiany (dodanie, zmodyfikowanie, usunięcie encji). Mechanizm ten dba o zachowanie spójności przechowywanych danych. Przykładowo modyfikacja wartości właściwości nawigującej wymusza automatyczną korektę wartości klucza obcego (i na odwrót – zmiana wartości klucza obcego automatycznie wymusza skorygowanie wartości właściwości nawigującej). Mechanizm śledzenia encji jest automatyczny i wbudowany w platformę `EF`. Dotyczy zarówno bezpośrednich instancji klas `POCO` (półautomatyczne śledzenie) jak i instancji dynamicznie wygenerowanych klas pośredników, czyli pochodnych po klasach `POCO` (w pełni automatyczne śledzenie).
Klasy dynamicznie wygenerowanych pośredników (klasy pochodne po klasach `POCO`) mają wbudowaną automatyczną obsługę śledzenia encji w ramach przeciążanych implementacji właściwości. Każda zmiana wartości właściwości wiążę się z natychmiastową korektą powiązanych wartości. Dlatego właściwości klas `POCO` muszą być wirtualne, aby można było je nadpisać.
Dla klas `POCO` zmiany są realizowane półautomatycznie. Analiza zmian odbywa się w oparciu o migawki danych (ang. `snapshot change tracking`), za pomocą których porównywane są aktualne wartości z oryginalnymi wartościami. Analiza nie odbywa się na bieżąco, ale jest inicjowana przy okazji wywołań niektórych metod:

* Korzystania z metod kolekcji obiektów-encji: `Find`, `Local`, `Add`, `Attach`, `Remove`
* Korzystania z metod kontekstu: `SaveChanges`, `GetValidationResult`, `Entry`
* Korzystania z metody obiektu `ChangeTracker: Entries`

Analizę zmian można też ręcznie zainicjować metodą kontekstu `DetectChanges`.
Mechanizm śledzenia encji jest kosztowny. Dlatego można go tymczasowo wyłączyć zmieniając ustawienia konfiguracji kontekstu:
```c#
mojeSklepy.Configuration.AutoDetectChangesEnabled = false;

```

Można też wymusić używanie tylko śledzenia metodą migawek wyłączając możliwość tworzenia dynamicznych klas pochodnych po klasach `POCO`. Uwaga wtedy wyłączone zostanie też "leniwe ładowanie".
```c#
mojeSklepy.Configuration.ProxyCreationEnabled = false;

```

Mechanizm śledzenia encji oferuje teżdostęp do wszystkich śledzonych obiektów. Każdy z obiektów encji ma:

* Określony stan (`Detached`, `Unchanged`, `Added`, `Deleted`, `Modified`) analogicznie jak w wierszach – obiektach klasy `DataRow`
* Kopię wartości oryginalnych i obecnych

Wszystkie te wartości można odczytać:
```c#
foreach (DbEntityEntry entry in mojeSklepy.ChangeTracker.Entries())
{
    string originalValues, currentValues;

    if (entry.State == EntityState.Added) { originalValues = null; }
    else { originalValues = GetValuesAsString(entry.OriginalValues); }

    if (entry.State == EntityState.Deleted) { currentValues = null; }
    else { currentValues = GetValuesAsString(entry.CurrentValues); }

    Console.WriteLine("{0} ({1}): {2}==>>{3}", entry.Entity, entry.State, 
                      originalValues, currentValues);
}

```

Jest dopuszczalne ustawienie (zmienienie) stanu encji w konkretnym kontekście. Można przykładowo zaznaczyć do skasowania encję, której jeszcze nie ma w obecnym kontekście, bo jeszcze nie została odczytana z bazy danych:
```c#
KategoriaProduktu kat = new KategoriaProduktu
{
    Id = 10,
    Nazwa = "Statki kosmiczne"
};
mojeSklepy.Entry(kat).State = EntityState.Deleted;

```

Modyfikacja encji w ramach obcego kontekstu – wersja alternatywna z użyciem metod mechanizmu śledzenia wersji:
```c#
Produkt produkt;
using (MojeSklepy mojeSklepy = new MojeSklepy())
{
    produkt = mojeSklepy.Produkty.FirstOrDefault(p => p.Nazwa.StartsWith("Traban"));
    if (produkt != null)
    {
        // Znaleziono produkt do zmodyfikowania
        mojeSklepy.Entry(produkt).State = EntityState.Detached;
    }
}
if (produkt != null)
{
    // Modyfikacja poza kontekstem
    produkt.Nazwa = "Zmodyfikowana nazwa";
    produkt.Opis = "Zmodyfikowany opis";
    using (MojeSklepy mojeSklepy = new MojeSklepy())
    {
        // Zapis zmodyfikowanej encji do bazy danych w obcym kontekście
        mojeSklepy.Entry(produkt).State = EntityState.Modified;
        mojeSklepy.SaveChanges();
    }
}

```

### Walidacja encji

Encje przed uruchomieniem zapisu do bazy danych podlegają walidacji. Proste reguły walidacji można określić definiując jawnie model `EDM`, korzystając z adnotacji `Data Annotations` lub z interfejsu `Fluent API`. Dodatkowo istnieje możliwość samodzielnego zaprogramowania bardziej złożonych reguł walidacyjnych. Odbywa się to w ramach nadpisanej metody `ValidateEntity` kontekstu. Zdefiniowana m.in. tam walidacja jest realizowana w trakcie wywołania metody `SaveChanges` kontekstu. Zapis do bazy danych jeszcze przed jego rozpoczęciem może zakończyć się wyjątkiem `DbEntityValidationException`, jeśli walidacja zgłosi błędy. Domyślnie walidacja dotyczy tylko nowych i modyfikowanych encji.
```c#
protected override DbEntityValidationResult ValidateEntity(
    DbEntityEntry entityEntry, IDictionary<object, object> items)
{
    var results = base.ValidateEntity(entityEntry, items);
    if (entityEntry.Entity is KategoriaProduktu)
    {
        KategoriaProduktu kat = (KategoriaProduktu)entityEntry.Entity;
        if (kat.Kod.Grupa.StartsWith("X"))
        { 
            results.ValidationErrors.Add(
            new DbValidationError("Kod.Grupa", "Nieprawidłowa grupa kodu kategorii. " + 
                "Niedozwolone są grupy rozpoczynające się od znaku X."));
        }
        if (kat.Nazwa.StartsWith(" ") || kat.Nazwa.EndsWith(" "))
        {
            results.ValidationErrors.Add(
            new DbValidationError("Nazwa", 
                "Nazwa kategorii nie można zaczynać/kończyć się spacją."));
        }
    }
    return results;
}

```

W przypadku wystąpienia błędów walidacji zgłaszany jest wyjątek typu `DbEntityValidationException`. Obiekt tego wyjątku zawiera zagregowane błędy walidacji. Są one pogrupowane względem encji, której dotyczą. Każdy błąd zawiera informację, której właściwości dotyczy, oraz treść komunikatu wyjaśniającego przyczynę błędu.
```c#
try
{
    mojeSklepy.SaveChanges();
}
catch (DbEntityValidationException exc)
{
    foreach (DbEntityValidationResult entValErr in exc.EntityValidationErrors)
    {
        // Informacje o błędnej encji
        Console.WriteLine("Encja: {0} ({1}) - {2}", entityErr.Entry.Entity, 
                          entityErr.Entry.State, entityErr.IsValid);

        foreach (DbValidationError error in entityErr.ValidationErrors)
        {
            // Szczegóły błędu
            Console.WriteLine("- Błąd: PropertyName={0}; ErrorMessage: {1}", 
                              error.PropertyName, error.ErrorMessage);
        }
    }
}

```

Listę błędów konkretnej encji lub całego kontekstu można uzyskać w dowolnym momencie działania programu. Należy w tym celu wywołać metodę `GetValidationResult` kontekstu, jeśli chcemy uzyskać błędy walidacji wszystkich encji w kontekście. Jeśli chcemy przeprowadzić walidację tylko jednej encji, to należy skorzystać z klasy `DbEntityEntry`.
```c#
// Błędy walidacji konkretnej encji
var validationResults = mojeSklepy.Entry(encja).GetValidationResult();
PrintValidationResults(validationResults);

// Wszystkie błędy walidacji w kontekście
PrintValidationResults(mojeSklepy.GetValidationErrors());

```

## Wiązanie danych w aplikacjach WPF

Wiązanie danych (ang. `data binding`) to mechanizm, który umożliwia automatyczny odczyt danych ze źródła danych i przypisanie odczytanych wartości wybranym właściwościom kontrolek. Wiązanie danych może być dwustronne, co oznacza, że możliwe jest automatyczne uaktualnianie źródła danych na podstawie zmian zaistniałych w kontrolkach.
Aby powiązać kontrolki `WPF` (ang. `Windows Presentation Foundation`) z danymi pochodzącymi z platformy `EF` należy jako źródło danych użyć właściwości `Local` kolekcji obiektów-encji. Wcześniej oczywiście należy "załadować" lokalny cache kolekcji encji danymi z bazy danych:
```c#
MojeSklepy mojeSklepy = new MojeSklepy();
private void Window_Loaded(object sender, RoutedEventArgs e)
{ 
    mojeSklepy.Produkty.Load();
    KontrolkaWpf.Source = mojeSklepy.Produkty.Local;
}

```

Jako kontrolkę służąca do wyświetlania i edycji danych można użyć np. kontrolkę `DataGrid`.
```xml
<DataGrid x:Name="KontrolkaWpf" HorizontalAlignment="Left" Margin="0" 
            VerticalAlignment="Top" AutoGenerateColumns="False">
    <DataGrid.Columns>
        <DataGridTextColumn Header="ID" Binding="{Binding Path=Id}" />
        <DataGridTextColumn Header="Nazwa" Binding="{Binding Path=Nazwa}"/>
        <DataGridTextColumn Header="Opis" Binding="{Binding Path=Opis}"/>
        <DataGridTextColumn Header="Kategoria" 
                            Binding="{Binding Path=KategoriaProduktu.Nazwa}"/>
    </DataGrid.Columns>
</DataGrid>

```

Do powiązania poszczególnych kolumn z konkretnymi właściwościami należy użyć atrybutu `Binding`, np:
```c#
Binding="{Binding Path=Id}"

```

Można też używać właściwości nawigujących:
```c#
Binding="{Binding Path=KategoriaProduktu.Nazwa}"

```

Wiązanie danych z platformą `EF` w kontrolce `DataGrid`:
![image](_images/ef/data-binding-wpf.*)

Ponieważ kontrolka `DataGrid` domyślnie realizuje dwustronne wiązanie danych, zmiany w polach kontrolki `DataGrid` są automatycznie odzwierciedlane w kolekcji encji. Zapis zmian do bazy danych realizowany jest już klasycznie przez kontekst z platformy `EF`.
```c#
private void Button_Click(object sender, RoutedEventArgs e)
{
    mojeSklepy.SaveChanges();
}

```
