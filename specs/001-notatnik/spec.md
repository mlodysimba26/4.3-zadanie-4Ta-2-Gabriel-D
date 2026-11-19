# Specyfikacja: Podręczny Notatnik Osobisty (Notatnik-v2)

Dokument określa wymagania funkcjonalne, ograniczenia zakresowe oraz kryteria akceptacji dla modułu notatnika osobistego tworzonego w architekturze Spec-Driven Development.

---

## 1. Cel

Stworzenie minimalistycznej, w pełni lokalnej aplikacji biurkowej do natychmiastowego zapisywania i przeglądania podręcznych notatek tekstowych. Program ma umożliwiać utrwalenie myśli w czasie poniżej kilku sekund, zapisując dane w czytelnym dla człowieka formacie Markdown z zachowaniem odwróconej chronologii (najnowsze wpisy na początku), bez potrzeby połączenia z siecią i bez uzależnienia od zamkniętych formatów bazodanowych.

---

## 2. Dla kogo

- **Użytkownik docelowy:** Osoba pracująca lokalnie na pojedynczym komputerze (deweloper, uczeń, pracownik biurowy), potrzebująca szybkiego notatnika do codziennych zapisków, list zadań i notatek podręcznych.
- **Charakterystyka potrzeb:** Ceni natychmiastową gotowość narzędzia (uruchomienie poniżej dwóch sekund), prywatność (dane nie opuszczają urządzenia), przejrzysty interfejs w języku polskim oraz bezpośredni dostęp do swoich danych w zewnętrznych edytorach tekstu. Nie potrzebuje kont użytkowników, logowania, pracy grupowej ani chmurowej synchronizacji.

---

## 3. Co robi

- Prezentuje prosty, jednoekranowy interfejs graficzny w języku polskim.
- Oferuje jednoliniowe pole tekstowe z tekstem zastępczym na wpisanie tytułu notatki.
- Oferuje wieloliniowe pole tekstowe z tekstem zastępczym na wpisanie treści notatki.
- Udostępnia przycisk „Zapisz notatkę”, który utrwala notatkę w pliku magazynu danych na dysku.
- Blokuje przycisk zapisu (stan nieaktywny), gdy pole tytułu jest puste lub zawiera wyłącznie białe znaki.
- Formatuje każdy wpis w standardzie Markdown z nagłówkiem poziomu drugiego: `## YYYY-MM-DD HH:mm — Tytuł`, po którym następuje pusty wiersz, treść notatki i kolejny pusty wiersz oddzielający wpisy.
- Zapisuje wpisy w porządku odwróconej chronologii — każda nowo utworzona notatka wstawiana jest na samym początku pliku magazynu, powyżej starszych notatek.
- Zapisuje i odczytuje wszystkie pliki w standardzie kodowania UTF-8, bezbłędnie zachowując polskie znaki diakrytyczne (`ąćęłńóśźżĄĆĘŁŃÓŚŹŻ`).
- Natychmiast po udanym zapisie czyści pola formularza oraz odświeża listę notatek w oknie aplikacji, umieszczając nowy wpis na samej górze.
- Wyświetla listę wszystkich dotychczas zapisanych notatek od najnowszej do najstarszej z widocznym tytułem i znacznikiem czasu.
- Po wybraniu elementu z listy prezentuje pełną treść notatki w widoku podglądu.
- Przy starcie programu automatycznie wczytuje istniejący plik magazynu i prezentuje listę wszystkich zapisanych dotąd notatek.
- W przypadku błędu wejścia/wyjścia (np. plik zablokowany, brak uprawnień) nie ulega awarii procesu, lecz wyświetla zrozumiały komunikat o błędzie po polsku na pasku statusu i zachowuje wpisany tekst w polach formularza.

---

## 4. Czego nie robi

- Nie łączy się z internetem, nie odpytuje zewnętrznych API ani nie wysyła telemetrii (zgodnie z zasadą 1 Konstytucji).
- Nie wymaga ani nie posiada systemu kont, logowania, profili, haseł ani ról (zgodnie z zasadą 3 Konstytucji).
- Nie umożliwia edycji ani usuwania pojedynczych historycznych notatek z poziomu interfejsu graficznego w tej wersji (ewentualne korekty użytkownik wykonuje bezpośrednio w otwartym pliku Markdown).
- Nie stosuje formatowania bogatego (WYSIWYG), kolorowania składni, osadzania tabel ani dołączania plików graficznych.
- Nie zawiera wyszukiwania pełnotekstowego, tagowania ani hierarchicznych folderów (decyzja o cięciu zakresu ADR 0001).
- Nie szyfruje pliku magazynu danych — zawartość pozostaje otwartym, czytelnym tekstem (zgodnie z zasadą 2 Konstytucji).
- Nie integruje się z usługami chmurowymi (Dropbox, Google Drive, OneDrive).

---

## 5. Kryteria akceptacji

### Kryterium 1: Poprawny zapis nowej notatki z formatem daty i tytułu
- **Sytuacja:** Użytkownik wpisuje tytuł „Plan na dzisiaj”, treść „1. Napisać specyfikację\n2. Zrobić commit” i klika przycisk „Zapisz notatkę”.
- **Wynik:** Nowa notatka zostaje zapisana w pliku magazynu z nagłówkiem `## YYYY-MM-DD HH:mm — Plan na dzisiaj` (gdzie data i godzina odpowiadają chwili zapisu), pustą linią, zadaną treścią i kolejną pustą linią. Pola formularza w oknie zostają wyczyszczone, a lista notatek natychmiast pokazuje nowy wpis na samej górze.
- **Sposób sprawdzenia:** Uruchom program, wprowadź powyższy tytuł i treść, naciśnij „Zapisz notatkę”. Zweryfikuj, czy pola edycyjne stały się puste, a pierwszy wiersz listy w programie pokazuje bieżący czas i tytuł. Otwórz plik magazynu w edytorze tekstowym i sprawdź obecność nagłówka `## ` z dokładną datą i separatorem `—`.

### Kryterium 2: Blokada zapisu przy pustym tytule
- **Sytuacja:** Pole tytułu notatki jest puste lub zawiera wyłącznie białe znaki (spacje, tabulacje).
- **Wynik:** Przycisk „Zapisz notatkę” pozostaje nieaktywny (`IsEnabled = false`), kliknięcie w niego nie wywołuje żadnego efektu, a żaden wpis nie zostaje dodany do magazynu ani listy.
- **Sposób sprawdzenia:** Uruchom aplikację. Sprawdź stan przycisku zapisu bez wpisywania tekstu. Następnie wprowadź do pola tytułu trzy spacje i sprawdź, czy przycisk nadal pozostaje nieaktywny. Wprowadź dowolną treść w pole treści przy pustym tytule — upewnij się, że przycisk wciąż jest zablokowany.

### Kryterium 3: Odwrócona chronologia zapisów (najnowszy wpis na początku)
- **Sytuacja:** W magazynie danych istnieją już wcześniej utrwalone notatki, a użytkownik dodaje kolejny wpis.
- **Wynik:** Nowo utworzona notatka zostaje wstawiona na sam początek pliku magazynu (przed wszystkimi wcześniejszymi wpisami) oraz pojawia się na pozycji 0 (na szczycie) listy w interfejsie graficznym.
- **Sposób sprawdzenia:** Dodaj notatkę o tytule „Pierwsza”. Następnie dodaj notatkę o tytule „Druga”. Otwórz plik magazynu na dysku i zweryfikuj, że sekcja z notatką „Druga” znajduje się fizycznie wyżej w pliku niż sekcja „Pierwsza”. W oknie programu sprawdź, czy notatka „Druga” jest na pierwszym miejscu listy.

### Kryterium 4: Bezbłędna obsługa polskich znaków diakrytycznych (UTF-8)
- **Sytuacja:** Użytkownik wprowadza polskie litery diakrytyczne (`ąćęłńóśźż ĄĆĘŁŃÓŚŹŻ`) w tytule i treści notatki (np. „Zażółć gęślą jaźń”) i zatwierdza zapis.
- **Wynik:** Wszystkie polskie litery zostają utrwalone w magazynie danych w kodowaniu UTF-8 i są wyświetlane w oknie programu bez zniekształceń, znaków zapytania czy błędnych sekwencji bajtów.
- **Sposób sprawdzenia:** Wpisz notatkę z tytułem i treścią zawierającą komplet polskich znaków diakrytycznych. Zapisz notatkę, zamknij program, po czym uruchom go ponownie. Zweryfikuj poprawność wyświetlania znaków na liście i w podglądzie w oknie programu oraz po otwarciu pliku magazynu w zewnętrznym edytorze z kodowaniem UTF-8.

### Kryterium 5: Trwałość danych i odczyt po ponownym uruchomieniu programu
- **Sytuacja:** Użytkownik zapisał kilka notatek, zamknął aplikację, a następnie uruchomił ją ponownie.
- **Wynik:** Program startuje w czasie krótszym niż dwie sekundy i od razu wczytuje wszystkie dotychczasowe notatki z pliku magazynu, prezentując pełną listę z zachowaniem odwróconej chronologii.
- **Sposób sprawdzenia:** Zapisz trzy zróżnicowane notatki w aplikacji. Zamknij proces programu. Uruchom program ponownie, mierząc czas do pełnej gotowości interfejsu (poniżej 2 sekund). Upewnij się, że lista zawiera dokładnie trzy elementy w prawidłowej kolejności od najnowszej do najstarszej.

### Kryterium 6: Bezpieczeństwo zapisu w przypadku awarii wejścia/wyjścia
- **Sytuacja:** Podczas próby zapisu notatki występuje błąd wejścia/wyjścia na dysku (np. plik magazynu ma nadany atrybut „tylko do odczytu” lub proces nie ma praw zapisu).
- **Wynik:** Aplikacja nie ulega awarii procesu (brak nagłego zamknięcia okna), na pasku stanu pojawia się czytelny komunikat błędu w języku polskim, a wprowadzony tekst w polach tytułu i treści NIE zostaje utracony, umożliwiając ponowienie próby po usunięciu blokady.
- **Sposób sprawdzenia:** Wprowadź tytuł i treść notatki do programu. W wierszu poleceń nadaj plikowi magazynu atrybut tylko do odczytu (`attrib +r`). W aplikacji kliknij przycisk „Zapisz notatkę”. Sprawdź, czy program nadal działa, czy w polu statusu pojawił się komunikat o błędzie zapisu oraz czy wpisany tekst pozostał w polach formularza. Usuń atrybut (`attrib -r`) i kliknij „Zapisz” ponownie — notatka powinna zostać pomyślnie zapisana.

---

## 6. Do ustalenia

- **Zewnętrzne modyfikacje pliku magazynu:** Czy jeśli użytkownik zmodyfikuje zawartość pliku magazynu w zewnętrznym programie (np. Notatniku) w trakcie działania aplikacji, program powinien automatycznie wykrywać zmiany i odświeżać listę, czy odczyt ma następować jedynie przy starcie i własnych zapisach?
- **Zapis notatki z pustą treścią:** Czy dozwolony jest zapis notatki, która posiada poprawny tytuł, lecz pole treści jest całkowicie puste (notatka jednolinijkowa), czy oba pola muszą być wypełnione?
- **Odporność parsera na znaczniki Markdown w treści:** W jaki sposób program powinien zabezpieczać treść notatki, jeśli użytkownik wpisze w niej wiersz zaczynający się od `## ` (nagłówek poziomu drugiego w Markdown), aby parser nie zinterpretował go jako początku nowego wpisu?
- **Zabezpieczenie przed uszkodzeniem pliku przy nagłej awarii (zapis atomowy):** Czy zapis do pliku magazynu powinien odbywać się poprzez plik tymczasowy z atomową zamianą (`File.Replace`), aby wyeliminować ryzyko utraty danych w przypadku nagłego zaniku zasilania w trakcie operacji zapisu?
- **Zachowanie przy braku lub uszkodzeniu pliku magazynu:** Czy przy pierwszym uruchomieniu, gdy plik magazynu jeszcze nie istnieje, program powinien utworzyć pusty plik od razu, czy dopiero przy pierwszym zapisie; oraz jak program powinien zareagować na uszkodzoną strukturę pliku (np. czy tworzyć automatyczną kopię `.bak`)?
