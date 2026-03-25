# Dokumentacja projektu dla AI — pliki CLAUDE.md

System hierarchicznych plików `CLAUDE.md` rozmieszczonych w projekcie, które pomagają AI coding agents (Claude Code, Cursor, itp.) rozumieć codebase bez czytania każdego pliku.

## Idea

AI agent wchodzący w projekt nie zna architektury, konwencji ani reguł biznesowych. Zamiast liczyć na to, że sam się domyśli, dajemy mu `CLAUDE.md` — zwięzłe pliki z kontekstem, rozmieszczone tam gdzie są potrzebne.

```
projekt/
├── CLAUDE.md              # overview: stack, architektura, konwencje
├── src/
│   ├── orders/
│   │   ├── CLAUDE.md      # domena zamówień: odpowiedzialność, pliki, reguły
│   │   └── ...
│   ├── payments/
│   │   ├── CLAUDE.md      # domena płatności
│   │   └── ...
│   └── shared/
│       ├── CLAUDE.md      # współdzielone utility i patterny
│       └── ...
```

Agent ładuje root `CLAUDE.md` na start, a potem doładowuje kontekst pakietu gdy wchodzi w dany katalog.

## Root CLAUDE.md

Wysoki poziom — overview i nawigacja, nie detale:

```markdown
# Nazwa Projektu

## Stack
- **Language:** TypeScript 5.x
- **Framework:** NestJS
- **Database:** PostgreSQL
- **Key libraries:** Prisma, Bull, Redis

## Architecture
Modular monolith z podziałem na domeny biznesowe.

## Project Structure
src/
├── orders/       # Zarządzanie zamówieniami
├── payments/     # Przetwarzanie płatności
├── shared/       # Współdzielone typy i utility

## Conventions
- **Files:** kebab-case
- **Functions:** camelCase
- **Tests:** *.spec.ts obok pliku źródłowego

## Key Patterns
- Repository Pattern: abstrakcja dostępu do danych
- Result<T, E>: explicit error handling bez wyjątków

## Getting Started
- Build: `npm run build`
- Test: `npm test`
- Run: `npm run start:dev`
```

**Rozmiar:** 50-100 linii. Jeśli jest dłuższy, przenosisz detale do CLAUDE.md pakietu.

## Package CLAUDE.md

Fokus na jednym module/domenie:

```markdown
# Orders

## Purpose
Zarządzanie cyklem życia zamówienia — tworzenie, walidacja, zmiana statusu.

## Key Concepts
- **Order:** agregat z pozycjami i statusem
- **OrderStatus:** maszyna stanów (draft → confirmed → shipped → delivered)

## Main Components
| File | Responsibility |
|------|----------------|
| `OrderService.ts` | Logika biznesowa zamówień |
| `OrderRepository.ts` | Dostęp do danych (Prisma) |
| `OrderController.ts` | HTTP endpoints |

## Dependencies
- **Internal:** shared (Result type, validation utils)
- **External:** Prisma, Bull (async jobs)

## Patterns Used
- Repository Pattern → see `OrderRepository.ts`
- Result Monad → see `shared/Result.ts`

## Important Rules
- Zamówienie nie może zmienić statusu wstecz
- Usunięcie zamówienia = soft delete (pole deletedAt)
```

**Rozmiar:** 30-60 linii.

## Gdzie umieszczać CLAUDE.md

**Tak:**
- Root projektu (zawsze)
- Pakiety domenowe (`src/orders/`, `src/payments/`)
- Warstwy architektoniczne jeśli wyraźnie odseparowane (`src/api/`, `src/core/`)
- Współdzielone utility (`src/shared/`)
- Złożone subsystemy wymagające wyjaśnienia

**Nie:**
- Każdy folder (za granularnie)
- Foldery z 1-2 prostymi plikami
- Foldery testowe (chyba że mają złożone test utilities)
- Foldery z generowanym kodem

## Komendy do zarządzania

### `/tasks:ai-enable-project` — inicjalizacja

Skanuje codebase i tworzy pełną hierarchię `CLAUDE.md` od zera:

1. Analizuje stack, architekturę, strukturę pakietów, konwencje, testy
2. Identyfikuje które foldery potrzebują CLAUDE.md
3. Generuje root CLAUDE.md + pliki pakietowe

**Użycie:** jednorazowo przy onboardingu projektu na AI-assisted development.

### `/tasks:ai-update-project` — aktualizacja

Audytuje istniejące `CLAUDE.md` i synchronizuje je z aktualnym stanem kodu:

1. **Wykrywa drift** — pliki wymienione w dokumentacji które już nie istnieją, zmienione nazwy, nowe zależności
2. **Znajduje luki** — nowe pakiety bez CLAUDE.md, nowe patterny warte udokumentowania
3. **Aktualizuje** — poprawia ścieżki, dodaje nowe komponenty, aktualizuje sekcję "Last Verified"
4. **Raportuje** — generuje change report ze wszystkimi zmianami

**Użycie:** okresowo, po większych zmianach w kodzie.

**Zasada:** kod wygrywa z dokumentacją. Jeśli CLAUDE.md mówi jedno a kod drugie — aktualizujemy CLAUDE.md, nie kod.

## Instalacja komend

Pliki komend są w katalogu [`commands/`](commands/) obok tego artykułu:

```bash
cp -r commands/tasks ~/.claude/commands/
```

Po skopiowaniu zrestartuj Claude Code — komendy pojawią się po wpisaniu `/tasks:ai-`.

## Dobre praktyki

- **Pisz dla AI, nie dla ludzi** — bądź faktyczny i precyzyjny, pomiń narrację
- **WHAT i WHERE, nie WHY** — agent potrzebuje wiedzieć co gdzie jest, nie historię decyzji
- **Konkretne ścieżki** — `src/orders/OrderService.ts`, nie "serwis zamówień"
- **Patterny z referencją** — `Repository Pattern → see OrderRepository.ts:L12-48`
- **Każdy CLAUDE.md w swoim scope** — nie powtarzaj informacji z roota w pakiecie
