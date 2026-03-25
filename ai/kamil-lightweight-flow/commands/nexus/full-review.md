# Full Review (Nexus All-in-One)

Wykonaj PEŁNY review zmian w bieżącym BRANCHU (w porównaniu do main/master). Przejdź przez WSZYSTKIE 4 fazy po kolei.

## Krok 0: Przygotowanie
Użyj poniższego algorytmu żeby zebrać WSZYSTKIE zmiany — commitowane, staged i unstaged:

Ustal branch bazowy:

bash   git rev-parse --verify main 2>/dev/null && echo "main" || echo "master"

Pobierz PEŁNY diff (commitowane zmiany vs baza + lokalne nieskomitowane):

bash   # Zmiany commitowane na branchu vs main
   git diff main...HEAD
   # Zmiany staged (dodane przez git add, jeszcze nieskomitowane)
   git diff --cached
   # Zmiany unstaged (edytowane ale jeszcze nie git add)
   git diff
Połącz wyniki wszystkich trzech — to jest Twój PEŁNY obraz zmian.

Pobierz listę WSZYSTKICH zmienionych/nowych plików:

bash   # Pliki zmienione na branchu vs main
   git diff main...HEAD --name-only
   # Pliki staged
   git diff --cached --name-only
   # Pliki unstaged
   git diff --name-only
   # Pliki nowe, nieśledzone (untracked)
   git ls-files --others --exclude-standard
Połącz i zdeduplikuj tę listę.

Przeczytaj plik .nexus-rules.md z roota projektu (jeśli istnieje).
Jeśli jesteś NA BRANCHU main/master i nie ma diff vs HEAD — to znaczy że zmiany są tylko lokalne. Użyj git diff i git diff --cached jako jedyne źródło.

WAŻNE: Nowe pliki (untracked) nie pojawią się w żadnym git diff. Dla każdego pliku z git ls-files --others --exclude-standard przeczytaj CAŁY plik i traktuj go jako 100% nowy kod do review.

---

## FAZA 1: SEMANTIC REVIEW
Dla KAŻDEJ zmienionej funkcji/metody/komponentu napisz:
- **Co się zmieniło w zachowaniu** (nie "zmieniono linię 42", tylko "dodano walidację wieku")
- **Dlaczego to może być ryzykowne**
- **Blast radius** — jakie moduły/pliki mogą być dotknięte
- Lista kluczowych decyzji architektonicznych i pytania do autora

## FAZA 2: EDGE CASE HUNT
Dla każdej zmienionej funkcji sprawdź:
- Co się stanie dla: null, undefined, pusty string, pusta lista, -1, 0, MAX_INT, duplikaty?
- Off-by-one (> vs >=)?
- Brak obsługi błędów?
- Race conditions?
- Czy zmiana usuwa istniejące zabezpieczenia?
Dla każdego problemu podaj: scenariusz, problematyczna linia, fix w pseudokodzie.

## FAZA 3: RULES CHECK
(Pomiń tę fazę jeśli `.nexus-rules.md` nie istnieje — napisz tylko: "Brak pliku .nexus-rules.md, pomijam sprawdzanie reguł.")
Dla KAŻDEJ reguły z `.nexus-rules.md` sprawdź czy diff ją łamie. Bądź binarny: złamana albo nie.

## FAZA 4: GARBAGE COLLECTION
Sprawdź CAŁY plik (nie tylko diff) dla każdego zmienionego pliku:
- Nieużywane importy, zmienne, funkcje
- console.log / debugger / print() (poza testami)
- Zakomentowany kod (nie komentarze dokumentacyjne)
- TODO/FIXME dodane przez agenta
- Stare nazwy w komentarzach po refaktorze
- Puste bloki catch

---

## Format odpowiedzi:

```
# 📋 Full Review Report

## 1. Semantic Review
### Co się zmieniło:
- [plik:funkcja] — [opis zmiany w logice]

### Blast Radius:
- [zmiana → wpływ na inne moduły]

### Pytania do autora:
- [pytania]

---

## 2. Edge Cases
### 🔴 WYSOKIE RYZYKO:
1. [plik:linia] — Scenariusz: ... / Problem: ... / Fix: ...

### 🟡 ŚREDNIE RYZYKO:
1. [plik:linia] — ...

---

## 3. Rules Check
### ❌ Złamane: [lista]
### ✅ Spełnione: [lista]

---

## 4. Garbage Collection
### 🗑️ Do usunięcia: [lista z plik:linia]
### 🔍 Do sprawdzenia: [lista]

---

## Podsumowanie
- Krytyczne problemy: [liczba]
- Ryzyka do sprawdzenia: [liczba]
- Śmieci do usunięcia: [liczba]
- Złamane reguły: [liczba]
- Ogólna ocena: [1-2 zdania — czy ten diff jest gotowy do merge'a]
```

## Ważne:
- Przejdź przez WSZYSTKIE 4 fazy — nie pomijaj żadnej
- Bądź konkretny — podawaj pliki i linie
- Nie wymyślaj problemów na siłę — jeśli jest czysto, napisz że jest czysto
- Jeśli diff jest duży (>500 linii), skup się na najważniejszych zmianach i zaznacz co pominąłeś