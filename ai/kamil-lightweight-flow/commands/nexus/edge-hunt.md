# Edge Case Hunter

Przeanalizuj zmiany w bieżącym BRANCHU i znajdź SUBTELNE BŁĘDY — kod który kompiluje się, wygląda poprawnie, ale może zawieść w edge case'ach.

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
2. Dla każdej zmienionej funkcji/metody przeanalizuj:
   - Jakie dane wejściowe przyjmuje?
   - Co się stanie dla: null, undefined, pusty string, pusta lista, -1, 0, MAX_INT, duplikaty, concurrent access?
   - Czy są warunki brzegowe (off-by-one: > vs >=, < vs <=)?
   - Czy brakuje obsługi błędów (try/catch, .catch(), error callback)?
   - Czy jest race condition (async bez awaita, brak locka, cache invalidation)?
   - Czy zmiana usuwa jakiekolwiek istniejące zabezpieczenia (null checki, walidacje, rate limiting)?

3. Dla KAŻDEGO znalezionego problemu podaj:
   - Konkretny scenariusz który spowoduje bug
   - Która linia/warunek jest problematyczny
   - Jak powinien wyglądać fix (1-3 linie pseudokodu)

## Format odpowiedzi:

```
## Edge Case Report

### 🔴 WYSOKIE RYZYKO (prawdopodobny bug):
1. [plik:linia] — Scenariusz: [co się stanie gdy X]
   Problem: [dlaczego to nie zadziała]
   Fix: [pseudokod]

### 🟡 ŚREDNIE RYZYKO (podejrzane):
1. [plik:linia] — Scenariusz: [opis]
   Problem: [opis]

### 🟢 SPRAWDZONE (wyglądają OK):
- [lista rzeczy które przeanalizowałeś i uznałeś za bezpieczne]
```

## Ważne:
- NIE raportuj problemów stylistycznych, formatowania, nazewnictwa
- SKUP SIĘ TYLKO na logice i zachowaniu
- Bądź konkretny — "może się zepsuć" to za mało, podaj DOKŁADNY scenariusz
- Jeśli nic nie znalazłeś — napisz to, nie wymyślaj problemów na siłę
