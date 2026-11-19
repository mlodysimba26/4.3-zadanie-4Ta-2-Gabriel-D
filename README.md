# Notatnik-v2: Specyfikacja, inwarianty i rozbiór wymagań

Projekt repozytorium `notatnik-v2` w ramach modułu poświęconego specyfikacjom technicznym, inwariantom architektonicznym oraz analizie wymagań w procesie Spec-Driven Development (SDD).

---

## 1. Przepisanie zdań na kryteria akceptacji (Warunek 3)

Poniżej znajduje się pięć zdań przepisanych na precyzyjne kryteria akceptacji w standardowej formule **„Kiedy …, to …”** wraz z procedurą weryfikacji **„Sprawdzenie: …”** oraz analizą decyzji projektowych:

---

### Zdanie 1: „Program obsługuje polskie znaki.”

- **Kryterium:**  
  **Kiedy** użytkownik wprowadzi tekst zawierający polskie litery diakrytyczne (`ąćęłńóśźż ĄĆĘŁŃÓŚŹŻ`) w polu tytułu lub treści notatki i zatwierdzi operację zapisu, **to** w oknie programu (na liście notatek oraz w panelu podglądu) oraz w pliku magazynu na dysku wszystkie litery zostaną trwale zapisane i wyświetlone w kodowaniu UTF-8 bez jakichkolwiek zniekształceń, znaków zapytania czy błędnych sekwencji bajtów (*mojibake*).
- **Sprawdzenie:**  
  Wprowadź do formularza notatkę z tytułem „Zażółć gęślą jaźń” i treścią zawierającą wszystkie polskie litery małe i wielkie. Kliknij przycisk „Zapisz notatkę”, zrestartuj aplikację i sprawdź, czy litery są poprawnie renderowane na liście i w podglądzie. Otwórz plik magazynu w zewnętrznym edytorze z aktywnym kodowaniem UTF-8 i zweryfikuj poprawność znaków.
- **Czy wymaga decyzji:**  
  Nie wymaga nowej decyzji projektowej — wymóg stosowania standardu UTF-8 stanowi nienaruszalny inwariant systemu określony wprost w Zasadzie 2 Konstytucji (*Zero Lock-In*).

---

### Zdanie 2: „Lista notatek jest zawsze aktualna.”

- **Kryterium:**  
  **Kiedy** użytkownik zatwierdzi zapis nowej notatki w aplikacji, **to** lista notatek w interfejsie graficznym natychmiast odświeża się, umieszczając nowo utworzony wpis na samej górze (na pozycji 0) z poprawną datą i tytułem, bez konieczności ponownego uruchamiania programu.
- **Sprawdzenie:**  
  Wpisz tytuł i treść nowej notatki, naciśnij przycisk „Zapisz notatkę” i zweryfikuj, czy pierwszy wiersz listy w oknie natychmiast zmienił się na dodany wpis.
- **Jakiej decyzji wymaga i dlaczego nie da się przepisać bez niej:**  
  Sformułowanie „zawsze aktualna” jest niejednoznaczne w środowisku wielozadaniowym. Zgodnie z Zasadą 2 Konstytucji użytkownik ma prawo w dowolnym momencie edytować plik `notatki.md` w zewnętrznym edytorze tekstu. Powstaje kluczowa decyzja architektoniczna:
  - *Decyzja:* Czy aplikacja ma implementować ciągły nasłuch zdarzeń systemu plików (`FileSystemWatcher`) i automatycznie przeładowywać listę w czasie rzeczywistym przy zewnętrznej modyfikacji pliku, czy też lista ma być aktualizowana jedynie w odpowiedzi na akcje wewnątrz aplikacji (zapis) oraz przy starcie programu?
  - *Przeniesienie do specyfikacji:* Ponieważ ciągłe przeładowywanie pliku może powodować konflikty z edycją formularza w GUI, kwestię zakresu automatycznego odświeżania przeniesiono do sekcji **„Do ustalenia”** w `specs/001-notatnik/spec.md`.

---

### Zdanie 3: „Zapis jest bezpieczny.”

- **Kryterium:**  
  **Kiedy** podczas operacji utrwalania notatki wystąpi błąd wejścia/wyjścia (np. plik magazynu ma zablokowane uprawnienia zapisu, jest oznaczony jako tylko do odczytu lub zablokowany przez inny proces), **to** aplikacja nie ulega awarii procesu (nie wyłącza się), wyświetla zrozumiały komunikat o błędzie w języku polskim na pasku statusu, a wprowadzony tekst w polach tytułu i treści pozostaje nienaruszony w formularzu, chroniąc użytkownika przed utratą danych.
- **Sprawdzenie:**  
  W systemie operacyjnym odbierz uprawnienia zapisu do pliku magazynu (np. nadaj atrybut tylko do odczytu). Wprowadź tytuł oraz treść do formularza i kliknij przycisk „Zapisz notatkę”. Sprawdź, czy okno aplikacji nadal działa, czy w polu statusu pojawił się komunikat błędu oraz czy pola formularza zachowały wpisaną treść. Następnie przywróć prawa zapisu i zapisz ponownie — operacja powinna zakończyć się sukcesem.
- **Jakiej decyzji wymaga i dlaczego nie da się przepisać bez niej:**  
  Pojęcie „bezpieczny zapis” jest skrajnie szerokie i nieprecyzyjne. Może oznaczać:
  1. *Bezpieczeństwo przed awarią:* obsługę wyjątków wejścia/wyjścia (`try-catch`), aby program się nie wyłączał.
  2. *Bezpieczeństwo integralności danych (zapis atomowy):* zapobieganie zniszczeniu całego archiwum dotychczasowych notatek, jeśli w połowie przepisywania pliku nastąpi nagły zanik zasilania (wymaga zapisu do pliku tymczasowego i atomowej zamiany).
  3. *Bezpieczeństwo kryptograficzne:* szyfrowanie pliku na dysku (co stałoby w sprzeczności z Zasadą 2 Konstytucji o jawności danych).
  - *Decyzja:* Bezpieczeństwo zdefiniowano na poziomie odporności na błędy I/O i nienaruszalności formularza. Decyzję dotyczącą wdrożenia mechanizmu zapisu atomowego przeniesiono do sekcji **„Do ustalenia”** w `specs/001-notatnik/spec.md`.

---

### Zdanie 4: „Pusta notatka nie jest zapisywana.”

- **Kryterium:**  
  **Kiedy** pole tytułu notatki pozostaje puste lub zawiera wyłącznie białe znaki (spacje, tabulacje, znaki nowego wiersza), **to** przycisk zapisu jest zablokowany i nieaktywny dla użytkownika, a próba jego wywołania nie dodaje żadnego wpisu do listy ani nie modyfikuje pliku magazynu na dysku.
- **Sprawdzenie:**  
  Uruchom aplikację. Sprawdź, czy przycisk zapisu jest zablokowany przy pustych polach. Wpisz w pole tytułu same spacje i zweryfikuj, czy przycisk nadal pozostaje nieaktywny. Wpisz treść do pola treści przy pustym tytule i upewnij się, że przycisk wciąż jest zablokowany.
- **Jakiej decyzji wymaga i dlaczego nie da się przepisać bez niej:**  
  Zdanie nie precyzuje, co decyduje o „pustości” notatki złożonej z dwóch pól (tytuł i treść):
  - *Decyzja:* Czy dopuszczamy utworzenie notatki jednolinijkowej, która ma niepusty tytuł, ale pustą treść (np. krótkie przypomnienie „Kupić chleb”), czy też oba pola muszą być jednocześnie wypełnione? W specyfikacji przyjęto, że krytyczny jest tytuł (ponieważ tworzy nagłówek Markdown `## ...`), natomiast kwestię dopuszczalności pustego pola treści przeniesiono do sekcji **„Do ustalenia”** w `specs/001-notatnik/spec.md`.

---

### Zdanie 5: „Program działa po ponownym uruchomieniu.”

- **Kryterium:**  
  **Kiedy** użytkownik zamknie aplikację po utrwaleniu jednej lub wielu notatek, a następnie uruchomi ją ponownie, **to** program uruchamia się w czasie krótszym niż dwie sekundy i od razu poprawnie wczytuje wszystkie dotychczasowe notatki z pliku magazynu, prezentując pełną listę wpisów w odwróconej chronologii (od najnowszej do najstarszej).
- **Sprawdzenie:**  
  Utwórz trzy różne notatki w programie. Zamknij proces aplikacji. Uruchom program ponownie, mierząc czas do pojawienia się gotowego okna (poniżej 2 sekund). Upewnij się, że na liście widoczne są wszystkie trzy notatki w poprawnej kolejności, a kliknięcie w notatkę wyświetla jej pełną treść w panelu podglądu.
- **Jakiej decyzji wymaga i dlaczego nie da się przepisać bez niej:**  
  Zdanie zakłada idealny scenariusz powtórnego uruchomienia, pomijając stany brzegowe:
  - *Decyzja:* Jak program ma się zachować przy pierwszym uruchomieniu, gdy plik magazynu jeszcze w ogóle nie istnieje na dysku (czy utworzyć pusty plik natychmiast, czy dopiero przy pierwszym zapisie); oraz jak ma zareagować, jeśli plik magazynu istnieje, lecz jego zawartość została uszkodzona lub sformatowana niezgodnie ze składnią?
  - *Przeniesienie do specyfikacji:* Decyzje dotyczące tolerancji błędów składni pliku i inicjalizacji magazynu przeniesiono do sekcji **„Do ustalenia”** w `specs/001-notatnik/spec.md`.
