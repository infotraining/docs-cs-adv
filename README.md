# Zaawansowane programowanie w .NET Framework 4.5 - Jupyter Book

Ten katalog zawiera materiały szkoleniowe w formacie Jupyter Book.

## Struktura

- `myst.yml` - Konfiguracja książki (MyST format)
- `index.md` - Strona główna
- `*.md` - Pliki MyST Markdown z treścią rozdziałów
- `_images/` - Katalog z obrazami
- `_static/custom.css` - Niestandardowy plik CSS (czcionka Fira Sans)
- `convert_rst.py` - Skrypt konwersji RST do MyST

## Budowanie książki

### Instalacja zależności

```bash
pip install -r requirements.txt
```

### Budowanie HTML

```bash
jupyter-book build .
```

Lub z katalogu głównego:

```bash
jupyter-book build jbook_pl
```

Alternatywnie użyj skryptu PowerShell:

```powershell
.\build.ps1 build   # Zbuduj książkę
.\build.ps1 serve   # Zbuduj i otwórz w przeglądarce
.\build.ps1 clean   # Wyczyść katalog build
```

### Wyświetlanie

Po zbudowaniu, otwórz plik:
```
jbook_pl\_build\html\index.html
```

### Czyszczenie

```bash
jupyter-book clean .
```

## Zawartość

1. **LINQ** - Language INtegrated Query
2. **Threads** - Programowanie współbieżne i wątki
3. **Parallel Extensions** - TPL i PLINQ
4. **Reflection** - Refleksje i metadane
5. **Entity Framework** - ORM dla .NET
6. **Security** - Bezpieczeństwo w .NET Framework
7. **Globalization** - Globalizacja aplikacji

## Uwagi

- Wszystkie pliki zostały przekonwertowane z reStructuredText (.rst) do MyST Markdown (.md)
- MyST to rozszerzenie Markdown z dodatkowymi możliwościami dla dokumentacji technicznej
- Obrazy znajdują się w katalogu `_images/`
- Konfiguracja jest w pliku `myst.yml` (nowy format Jupyter Book)
- **Czcionka**: Użyto czcionki Fira Sans dla tekstu i Fira Code/Fira Mono dla kodu
- Plik CSS: `_static/custom.css` zawiera wszystkie dostosowania wizualne
