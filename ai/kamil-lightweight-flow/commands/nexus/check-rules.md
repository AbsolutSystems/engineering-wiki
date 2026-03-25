# Check Project Rules

Przeanalizuj zmiany w bieżącym BRANCHU pod kątem zgodności z regułami projektu zdefiniowanymi w pliku `.nexus-rules.md` w katalogu głównym repozytorium.

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

2. Dla KAŻDEJ reguły z pliku `.nexus-rules.md` sprawdź czy diff ją łamie.
3. Raportuj wyniki.

## Format `.nexus-rules.md` (podaj użytkownikowi jeśli nie istnieje):

```markdown
# Reguły Projektu

## Wymagane biblioteki
- API calls: używaj WYŁĄCZNIE Orvala (nie axios, nie fetch bezpośrednio)
- State management: Zustand (nie Redux, nie Context dla globalnego stanu)
- Formularze: React Hook Form + Zod

## Zabronione wzorce
- Żadnych `any` w TypeScript
- Żadnych `console.log` w PRODUKCIE (tylko w plikach *.test.*)
- Żadnych bezpośrednich importów między modułami (używaj public API modułu)

## Architektura
- Struktura: Vertical Slices (każdy feature w swoim katalogu)
- Każdy nowy endpoint wymaga testu integracyjnego
- Zmiany w shared/ wymagają aktualizacji CHANGELOG.md

## Konwencje
- Nazwy komponentów: PascalCase
- Nazwy hooków: camelCase z prefixem "use"
- Nazwy plików: kebab-case
```

## Format odpowiedzi:

```
## Rules Check Report

### ❌ ZŁAMANE REGUŁY:
1. Reguła: [nazwa reguły]
   Gdzie: [plik:linia]
   Problem: [co dokładnie jest nie tak]
   Fix: [co trzeba zmienić]

### ✅ SPEŁNIONE REGUŁY:
- [lista reguł które sprawdziłeś i są OK]

### ⚠️ NIE DA SIĘ ZWERYFIKOWAĆ:
- [lista reguł których nie da się sprawdzić z samego diffa]
```

## Ważne:
- Sprawdzaj TYLKO reguły z .nexus-rules.md — nie dodawaj własnych opinii o kodzie
- Bądź binarny: reguła jest złamana albo nie. Bez "może" i "warto rozważyć"
- Jeśli reguła jest niejednoznaczna — raportuj to w sekcji "nie da się zweryfikować"
