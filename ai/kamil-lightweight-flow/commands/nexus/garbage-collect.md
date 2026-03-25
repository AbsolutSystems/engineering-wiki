# Garbage Collector (Leftovers Cleaner)

Przeskanuj zmiany w bieżącym BRANCHU i znajdź ŚMIECI zostawione przez agenta AI — rzeczy które powinny być usunięte ale zostały.

## Kroki:

1. Przygotowanie
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
2. Pobierz listę zmienionych plików: `git diff --name-only main...HEAD`
3. Dla KAŻDEGO zmienionego pliku sprawdź CAŁY plik (nie tylko diff) pod kątem:

### Co szukać:

**Martwy kod:**
- Nieużywane importy (zaimportowane ale nigdzie nie użyte w pliku)
- Nieużywane zmienne/stałe
- Zakomentowany kod (bloki `//` lub `/* */` które wyglądają na stary kod, nie na komentarze dokumentacyjne)
- Osierocone funkcje/metody (zdefiniowane ale nie wywoływane nigdzie w projekcie)
- Puste bloki catch (catch który nic nie robi)

**Śmieci debugowe:**
- `console.log` / `console.debug` / `print()` / `debugger` (poza plikami testowymi)
- TODO/FIXME/HACK komentarze dodane przez agenta w tym diffu
- Tymczasowe zmienne typu `temp`, `test`, `xxx`, `aaa`

**Niespójności po refaktorze:**
- Stare nazwy w komentarzach (komentarz mówi o `UserService` ale klasa została przemianowana na `UserRepository`)
- Stare typy/interfejsy które nie są już używane po zmianie
- Pliki testowe które testują usunięte funkcje

## Format odpowiedzi:

```
## Garbage Collection Report

### 🗑️ DO USUNIĘCIA:
1. [plik:linia] — [typ: martwy import / debug log / zakomentowany kod / ...]
   Powód: [dlaczego to jest śmieć]

### 🔍 DO SPRAWDZENIA (nie jestem pewien):
1. [plik:linia] — [opis] — może być celowe, ale wygląda jak leftover

### 📊 Podsumowanie:
- Znalezione śmieci: [liczba]
- Zmienione pliki: [liczba] / przeskanowane: [liczba]
- Czas od ostatniego czyszczenia: [jeśli da się ustalić z git log]
```

## Ważne:
- Sprawdzaj CAŁY plik, nie tylko zmienione linie — agent mógł zostawić śmieci w częściach których nie dotykał
- NIE raportuj celowych komentarzy dokumentacyjnych (JSDoc, docstring) jako śmieci
- NIE raportuj `console.log` w plikach testowych
- W razie wątpliwości — wrzuć do "do sprawdzenia", nie "do usunięcia"
