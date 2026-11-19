# Przegląd Specyfikacji: Podręczny Notatnik Osobisty (Notatnik-v2)

- **Rola recenzenta:** Niezależny recenzent architektury w procesie Spec-Driven Development (SDD) (agent w świeżej sesji).
- **Materiały poddane ocenie:** Wyłącznie pliki `konstytucja.md` oraz `specs/001-notatnik/spec.md`.

---

## 1. Proponowane drzewo plików projektu (wypisane przez czytelnika)

Na podstawie wymagań funkcjonalnych oraz inwariantów Konstytucji, programista przystępujący do implementacji utworzyłby następującą strukturę plików:

```text
NotatnikV2/
├── .gitignore
├── README.md                           # Dokumentacja projektu, wymagania środowiskowe i instrukcja uruchomienia
├── docs/
│   ├── konstytucja.md                  # Nienaruszalne inwarianty architektoniczne systemu
│   └── specs/
│       └── 001-notatnik/
│           ├── spec.md                 # Specyfikacja wymagań i kryteria akceptacji modułu
│           ├── plan.md                 # Plan techniczny i struktura kodu (izolacja od specyfikacji)
│           └── przeglad.md             # Raport z niezależnego przeglądu specyfikacji
│
├── src/
│   ├── NotatnikV2.Domain/              # Rdzeń biznesowy – niezależny od UI i systemu operacyjnego
│   │   ├── Models/
│   │   │   └── Note.cs                 # Model encji notatki (znacznik czasu, tytuł, treść, formatowany nagłówek)
│   │   └── Interfaces/
│   │       ├── INoteRepository.cs      # Kontrakt operacji składowania (odczyt, dopisanie na początek, obsługa błędów)
│   │       └── INoteMarkdownParser.cs  # Kontrakt parsera i serializatora formatu Markdown
│   │
│   ├── NotatnikV2.Infrastructure/      # Warstwa operacji wejścia/wyjścia na systemie plików
│   │   ├── Storage/
│   │   │   ├── FileNoteRepository.cs   # Implementacja zapisu UTF-8 z odwróconą chronologią i zapisem atomowym
│   │   │   └── MarkdownNoteParser.cs   # Parser wyodrębniający notatki z bloków '## YYYY-MM-DD HH:mm — Tytuł'
│   │   └── Common/
│   │       └── StorageConstants.cs     # Stałe formatu (separatory, domyślna nazwa pliku magazynu)
│   │
│   └── NotatnikV2.UI/                  # Warstwa prezentacji (interfejs graficzny)
│       ├── App.xaml / App.xaml.cs      # Punkt wejścia aplikacji, cykl życia, konfiguracja stylów
│       ├── Views/
│       │   ├── MainWindow.xaml         # Deklaracja jednoekranowego widoku: formularz, lista, podgląd, pasek statusu
│       │   └── MainWindow.xaml.cs      # Kod towarzyszący widoku (code-behind)
│       ├── ViewModels/
│       │   └── MainViewModel.cs        # Logika widoku: walidacja tytułu, stan przycisku zapisu, obsługa błędów I/O
│       └── Resources/
│           ├── Strings.pl-PL.xaml      # Wszystkie teksty interfejsu wyłącznie w języku polskim (zasada 5 Konstytucji)
│           └── Styles.xaml             # Style kontrolek zapewniające minimalistyczny i czytelny wygląd
│
└── tests/
    ├── NotatnikV2.Domain.Tests/
    │   └── MarkdownParserTests.cs      # Testy jednostkowe parsowania i serializacji (format daty, polskie znaki UTF-8, puste linie)
    ├── NotatnikV2.Infrastructure.Tests/
    │   └── FileNoteRepositoryTests.cs  # Testy integracyjne I/O (zapis na początku, odporność na brak praw zapisu, atomowość)
    └── NotatnikV2.UI.Tests/
        └── MainViewModelTests.cs       # Testy logiki prezentacji (blokada zapisu przy pustym tytule/białych znakach, czyszczenie pól)
```

---

## 2. Zidentyfikowane wycieki implementacyjne (Implementation Leaks)

W specyfikacji wymagań powinniśmy definiować **„co”** system robi i jakie są kryteria akceptacji z perspektywy użytkownika/kontraktu, a nie **„jak”** zostało to zakodowane w konkretnym języku, bibliotece lub systemie operacyjnym:

1. **Właściwość frameworka UI: `IsEnabled = false` (Kryterium 2)**
   - *Cytat ze specyfikacji:* „Przycisk „Zapisz notatkę” pozostaje nieaktywny (`IsEnabled = false`), kliknięcie w niego nie wywołuje żadnego efektu...”
   - *Dlaczego to wyciek:* `IsEnabled` to konkretna nazwa właściwości kontrolek w technologiach XAML/.NET (WPF, Avalonia, WinUI). W czystej specyfikacji zachowanie powinno być opisane funkcyjnie: *„przycisk zapisu jest zablokowany / wyłączony dla interakcji użytkownika”*, bez narzucania nazewnictwa właściwości z kodu.

2. **Narzucenie konkretnych poleceń powłoki Windows: `attrib +r` oraz `attrib -r` (Kryterium 6)**
   - *Cytat ze specyfikacji:* „W wierszu poleceń nadaj plikowi magazynu atrybut tylko do odczytu (`attrib +r`). [...] Usuń atrybut (`attrib -r`) i kliknij „Zapisz” ponownie...”
   - *Dlaczego to wyciek:* Jest to odwołanie do polecenia wiersza poleceń konkretnego systemu operacyjnego (Windows CMD). W procedurze weryfikacji powinno paść sformułowanie niezależne od środowiska, np.: *„Skonfiguruj plik magazynu tak, aby proces nie posiadał uprawnień do zapisu (np. atrybut tylko do odczytu lub blokada uprawnień w systemie plików)”*.

3. **Bezpośrednie wywołanie metody API .NET: `File.Replace` (Sekcja 6 – Do ustalenia)**
   - *Cytat ze specyfikacji:* „Czy zapis do pliku magazynu powinien odbywać się poprzez plik tymczasowy z atomową zamianą (`File.Replace`), aby wyeliminować ryzyko...”
   - *Dlaczego to wyciek:* `File.Replace` to nazwa statycznej metody z przestrzeni `System.IO` platformy .NET. W specyfikacji architektonicznej należy posługiwać się pojęciem domenowym/architektonicznym: *„zapis atomowy z wykorzystaniem pliku tymczasowego (ang. atomic file swap)”*, pozostawiając dobór konkretnego API programiście.

4. **Narzucenie rozszerzenia pliku kopii: `.bak` (Sekcja 6 – Do ustalenia)**
   - *Cytat ze specyfikacji:* „...czy tworzyć automatyczną kopię `.bak`”
   - *Dlaczego to wyciek:* Narzuca konkretną konwencję nazewniczą mechanizmu kopii zapasowej na poziomie kodu.

5. **Wewnętrzne identyfikatory projektowe: `ADR 0001` (Sekcja 4)**
   - *Cytat ze specyfikacji:* „(decyzja o cięciu zakresu ADR 0001)”
   - *Dlaczego to wyciek:* W specyfikacji wymagań nie powinny pojawiać się surowe odnośniki do wewnętrznych artefaktów zarządzania procesem wytwórczym bez wyjaśnienia ich kontekstu.

---

## 3. Pytania i wątpliwości (Luki i niejednoznaczności specyfikacji)

Zestawienie kwestii krytycznych dla dewelopera i testera, na które w pierwotnej treści specyfikacji brakuje jednoznacznej odpowiedzi:

### A. Format magazynu i obsługa danych na dysku
1. **Domyślna nazwa i lokalizacja pliku magazynu:** Gdzie fizycznie znajduje się plik? Czy to plik `notatki.md` w katalogu roboczym aplikacji, czy w dedykowanym katalogu danych użytkownika?
2. **Zachowanie przy pierwszym uruchomieniu:** Czy program tworzy pusty plik magazynu od razu po starcie, czy dopiero przy pierwszym zapisie notatki?
3. **Zapis atomowy a inwariant Konstytucji (Zasada 6 – Data Safety First):** Nadpisywanie całego pliku w celu umieszczenia notatki na początku bez zapisu atomowego stwarza ryzyko utraty całego archiwum w razie awarii zasilania. Zapis atomowy powinien być zdefiniowany jako wymóg bezpieczeństwa danych.
4. **Odporność na ręczne modyfikacje pliku:** Jak parser ma zareagować na plik, w którym użytkownik wstawił nieprawidłowy nagłówek lub uszkodził strukturę wpisu?

### B. Walidacja danych wejściowych i przypadki brzegowe
5. **Dopuszczalność notatki z pustą treścią:** Czy dozwolony jest zapis notatki, która posiada poprawny tytuł, lecz pole treści jest całkowicie puste (notatka jednolinijkowa)?
6. **Białe znaki i nowe linie w tytule:** Tytuł ma być jednoliniowy. Co się stanie, jeśli użytkownik wklei do pola tytułu tekst wielowierszowy? Czy aplikacja spłaszcza nowe linie do spacji, obcina tekst, czy blokuje zapis?
7. **Kolizja nagłówków Markdown w treści notatki:** Jeśli użytkownik wpisze w treści notatki wiersz zaczynający się od `## ` (np. `## Podrozdział`), jak parser odróżni go od nagłówka nowej notatki?
8. **Unikalność znacznika czasu:** Format czasu to `YYYY-MM-DD HH:mm`. Co jeśli użytkownik zapisze dwie notatki w ciągu tej samej minuty z takim samym tytułem?
9. **Warianty separatora tytułu:** W nagłówku zdefiniowano pauzę `—` (em-dash). Jeśli użytkownik dokona ręcznej edycji w edytorze i wpisze zwykły dywiz `-`, czy parser powinien to zaakceptować?

### C. Interfejs użytkownika i cykl życia
10. **Reakcja na zewnętrzne modyfikacje pliku:** Czy aplikacja powinna automatycznie odświeżać listę, gdy plik zostanie zmodyfikowany przez zewnętrzny edytor w trakcie jej działania?
11. **Stan początkowy widoku podglądu:** Co wyświetla się w panelu podglądu bezpośrednio po uruchomieniu aplikacji?
12. **Cykl życia komunikatów na pasku statusu:** Czy po udanym zapisie pojawia się komunikat sukcesu i kiedy pasek powraca do stanu spoczynku?
