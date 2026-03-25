# Semantic Review

Przeanalizuj zmiany w bieżącym BRANCHU (w porównaniu do main/master) i wygeneruj SEMANTYCZNY raport — nie tekstowy diff, ale opis zmian w logice i architekturze.

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
2. Dla KAŻDEJ zmienionej funkcji/metody/komponentu napisz:
   - **Co się zmieniło w zachowaniu** (nie "zmieniono linię 42", tylko "dodano walidację wieku użytkownika")
   - **Dlaczego to może być ryzykowne** (np. "brak obsługi wartości null", "zmiana typu może złamać kontrakt API")
   - **Jakie moduły/pliki mogą być dotknięte** tą zmianą (blast radius)
3. Na końcu daj podsumowanie:
   - Lista kluczowych decyzji architektonicznych podjętych w tym diffu
   - Lista pytań które reviewer powinien zadać autorowi (rzeczy niejasne, brakujące uzasadnienia)

## Format odpowiedzi:

```
## Raport Semantic Review

### Zmienione zachowania:
- [plik:funkcja] — opis zmiany w logice

### Ryzyka:
- [WYSOKIE/ŚREDNIE/NISKIE] — opis

### Blast Radius:
- Zmiana X wpływa na: moduł Y, moduł Z

### Pytania do autora:
- Dlaczego wybrano podejście X zamiast Y?

### Podsumowanie:
[2-3 zdania: co ten diff robi jako całość]
```

## Ważne:
- NIE opisuj zmian na poziomie tekstu ("dodano linię", "usunięto import")
- SKUP SIĘ na intencji i logice
- Jeśli widzisz zmianę której cel jest niejasny — powiedz to wprost
