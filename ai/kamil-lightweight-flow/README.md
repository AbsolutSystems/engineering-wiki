# Lightweight Flow — workflow Kamila z Claude Code

Lekki, modularny workflow do pracy z Claude Code oparty o trzy grupy slash komend: `/feature`, `/tasks` i `/nexus`. Każda odpowiada za inną fazę — od pomysłu, przez implementację, po review.

W odróżnieniu od GSD czy Superpowers nie jest to monolityczny system. To zestaw niezależnych komend, które można używać pojedynczo lub łączyć w pełny flow.

## Pełny flow

```
/feature → FEATURE.md → /tasks:generate-tasks → TASKS.md → /tasks:implement-tasks → /nexus:full-review
```

## 1. `/feature` — dyskusja o feature'ze

Przełącza Claude w tryb **Product Owner / Product Architect**. Claude nie pisze kodu — prowadzi rozmowę.

**Jak działa:**
1. **Zrozumienie potrzeby** — pyta o problem, użytkowników, oczekiwane zachowanie (2-3 pytania na start, nie 10)
2. **Analiza kodu** — przegląda codebase, szuka istniejących wzorców i modułów do wykorzystania
3. **Propozycja rozwiązań** — 2-3 alternatywy z tradeoffami i rekomendacją
4. **Doprecyzowanie** — iteracja na podstawie feedbacku, edge case'y, zależności
5. **Zapis** — generuje `FEATURE.md` z pełną specyfikacją

**Użycie:**
```
/feature nowy system notyfikacji
```

**Output:** `FEATURE.md` w roocie projektu — zawiera problem, wybrany approach, zakres (in/out of scope), plan implementacji, edge case'y.

**Kluczowa zasada:** Claude odmówi pisania kodu w tym trybie. To jest sesja planowania, nie implementacji.

## 2. `/tasks` — planowanie i implementacja

Dwie komendy tworzące pipeline: generowanie tasków i ich egzekucja.

### `/tasks:generate-tasks` — plan implementacji

Czyta `FEATURE.md` i codebase, produkuje `TASKS.md` — szczegółowy plan dla AI agenta.

**Co robi:**
- Analizuje architekturę, wzorce, konwencje, testy
- Pyta o dodatkowy kontekst i preferencje
- Rozbija feature na sekwencyjne taski z dokładnymi ścieżkami plików, sygnaturami metod i testami

**Format tasku:**
```markdown
### Task 3: Create ArchiveService
**Status:** pending

**Create:**
- `src/services/ArchiveService.ts`
- `src/services/ArchiveService.spec.ts`

**Requirements:**
- Implement `archive(content: Buffer, filename: string): Promise<ArchiveResult>`
- Use pattern from `src/services/ExistingService.ts`

**Tests:**
- Success: file archived, correct checksum
- Failure: SFTP error throws ArchiveException
- Edge: empty buffer handled gracefully
```

### `/tasks:implement-tasks` — egzekucja tasków

Czyta `TASKS.md` i implementuje taski jeden po drugim.

**Zasady:**
- Sekwencyjnie — każdy task buduje na poprzednim
- TDD gdzie ma sens (logika biznesowa, walidacja) — ale nie na siłę (config, scaffolding)
- Po zakończeniu zmienia status z `pending` na `completed`
- Jeśli znajdzie lepsze podejście — zatrzymuje się i pyta zamiast samodzielnie zmieniać plan

### Powiązane

Komendy `/tasks:ai-enable-project` i `/tasks:ai-update-project` do zarządzania dokumentacją CLAUDE.md opisane są w osobnym artykule: [Dokumentacja projektu dla AI](../claude-md-documentation/)

## 3. `/nexus` — review zmian

Zestaw komend do reviewowania kodu na bieżącym branchu (diff vs main).

### `/nexus:full-review` — pełny review (all-in-one)

Przechodzi przez 4 fazy:

1. **Semantic Review** — co się zmieniło w zachowaniu (nie "zmieniono linię 42", tylko "dodano walidację wieku"), blast radius, pytania do autora
2. **Edge Case Hunt** — null, pusty string, off-by-one, race conditions, brak error handlingu. Każdy problem z scenariuszem i fixem
3. **Rules Check** — sprawdzenie diff vs reguły z `.nexus-rules.md` (jeśli istnieje)
4. **Garbage Collection** — nieużywane importy, console.log, zakomentowany kod, puste catche, stale TODO

**Output:** raport z podsumowaniem — ile krytycznych problemów, ryzyk, śmieci, złamanych reguł i czy diff jest gotowy do merge'a.

### Pojedyncze fazy

Każdą fazę można odpalić osobno:

| Komenda | Faza |
|---|---|
| `/nexus:semantic-review` | Analiza semantyczna zmian |
| `/nexus:edge-hunt` | Szukanie edge case'ów |
| `/nexus:check-rules` | Sprawdzenie reguł projektu |
| `/nexus:garbage-collect` | Szukanie śmieci w kodzie |

## Kiedy używać czego

| Sytuacja | Komenda |
|---|---|
| Nowy feature, nie wiem jak podejść | `/feature` |
| Mam FEATURE.md, potrzebuję planu | `/tasks:generate-tasks` |
| Mam TASKS.md, czas kodować | `/tasks:implement-tasks` |
| Skończyłem, chcę review przed PR | `/nexus:full-review` |
| Szybki check edge case'ów | `/nexus:edge-hunt` |
| Nowy projekt, chcę dobre CLAUDE.md | [`/tasks:ai-enable-project`](../claude-md-documentation/) |

## Instalacja

Pliki komend są w katalogu [`commands/`](commands/) obok tego artykułu. Skopiuj je do `~/.claude/` zachowując strukturę:

```bash
# Całość
cp -r commands/* ~/.claude/commands/

# Lub wybiórczo, np. tylko nexus
cp -r commands/nexus ~/.claude/commands/
```

> **Uwaga:** `/feature` to skill, nie command — wymaga katalogu `~/.claude/skills/feature-discuss/SKILL.md`. Skopiuj go ręcznie:
> ```bash
> mkdir -p ~/.claude/skills/feature-discuss
> cp commands/feature-discuss.md ~/.claude/skills/feature-discuss/SKILL.md
> ```

### Struktura plików

```
commands/
├── feature-discuss.md              → ~/.claude/skills/feature-discuss/SKILL.md
├── tasks/
│   ├── generate-tasks-command.md   → ~/.claude/commands/tasks/
│   └── implement-tasks.md          → ~/.claude/commands/tasks/
└── nexus/
    ├── full-review.md              → ~/.claude/commands/nexus/
    ├── semantic-review.md          → ~/.claude/commands/nexus/
    ├── edge-hunt.md                → ~/.claude/commands/nexus/
    ├── check-rules.md              → ~/.claude/commands/nexus/
    └── garbage-collect.md          → ~/.claude/commands/nexus/
```

Po skopiowaniu zrestartuj Claude Code — komendy pojawią się po wpisaniu `/tasks:` lub `/nexus:`.

## Dlaczego nie GSD / Superpowers

- **Modularność** — każda komenda działa samodzielnie, nie musisz kupować całego systemu
- **Brak narzutu** — zero plików konfiguracyjnych, zero state management, zero instalacji
- **Kontrola** — sam decydujesz kiedy i co odpalić
- **Prostota** — 3 grupy komend zamiast framework'a do nauki
