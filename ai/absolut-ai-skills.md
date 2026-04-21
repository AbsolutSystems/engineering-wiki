# Absolut AI Skills

Plugin do Claude Code wspierający cały cykl developmentu — od dyskusji o feature'ze, przez planowanie i implementację, po code review.

Open-source repo: `AbsolutSystems/absolut-ai-skills`

## Instalacja

### 1. Dodaj marketplace

```bash
/plugin marketplace add AbsolutSystems/absolut-ai-skills
```

### 2. Zainstaluj plugin

```bash
/plugin install absolut-ai@absolut-ai-skills
```

### 3. Zweryfikuj

Zrestartuj Claude Code, wpisz `/absolut-ai:` — autocomplete powinien pokazać 6 skilli.

### Aktualizacja

```bash
/plugin install absolut-ai@absolut-ai-skills
```

## Pipeline developmentu

Każdy skill produkuje output konsumowany przez następny:

```
feature-discuss → generate-tasks → implement → review
     (CO?)           (JAK?)         (BUDUJ)    (SPRAWDŹ)
```

### `/absolut-ai:feature-discuss` — dyskusja o feature'ze

Interaktywna sesja Product Owner / Product Architect. Claude nie pisze kodu — prowadzi rozmowę o wymaganiach.

```bash
/absolut-ai:feature-discuss chce system powiadomien push dla uzytkownikow
```

- Zadaje pytania doprecyzowujące
- Analizuje codebase pod kątem istniejących wzorców i komponentów
- Proponuje 2-3 alternatywy z tradeoffami
- Iteruje na scope, edge case'ach, zależnościach

**Output:** `./absolut-ai/feature/planning-{slug}.md`, opcjonalnie `./docs/adr/YYYY-MM-DD-{slug}.md`

**Smart routing:** trywialne zmiany pomijają planning doc — sugeruje bezpośrednią implementację.

### `/absolut-ai:generate-tasks` — plan implementacji

Tworzy sekwencyjny plan z dokumentu planowania lub raportu review.

```bash
/absolut-ai:generate-tasks @absolut-ai/feature/planning-push-notifications.md
/absolut-ai:generate-tasks @absolut-ai/reviews/2026-04-21-feature-auth.md
```

- Czyta planning doc, `./absolut-ai/patterns.md`, `./absolut-ai/rules.md`, ADR-y
- Analizuje architekturę, wzorce, konwencje
- Produkuje sekwencyjne taski z dokładnymi ścieżkami plików, sygnaturami metod i testami

**Output:** `./absolut-ai/feature/tasks-{slug}.md`

### `/absolut-ai:implement` — egzekucja tasków

Implementuje taski sekwencyjnie z podejściem TDD.

```bash
/absolut-ai:implement @absolut-ai/feature/tasks-push-notifications.md
```

- Bierze pierwszy pending task, implementuje (testy najpierw)
- Aktualizuje status taska na `completed` in-place
- Tworzy ADR dla istotnych decyzji implementacyjnych

**Output:** kod + testy, zaktualizowany plik tasków

### `/absolut-ai:review` — pełny code review

4-fazowy review zmian na bieżącym branchu.

```bash
/absolut-ai:review
/absolut-ai:review develop    # custom base branch
```

| Faza | Fokus |
|------|-------|
| 1. Semantic Review | Zmiany w zachowaniu, blast radius, decyzje architektoniczne |
| 2. Edge Case Hunt | null, pusty string, off-by-one, race conditions |
| 3. Rules Check | Zgodność z `./absolut-ai/rules.md` |
| 4. Garbage Collection | Martwe importy, debug logi, zakomentowany kod |

**Output:** `./absolut-ai/reviews/YYYY-MM-DD-{branch-slug}.md`

**Smart follow-up:** 0-2 problemy — ręczny fix. 3+ problemów — sugeruje `/absolut-ai:generate-tasks` na raporcie review.

## Skille wspierające

### `/absolut-ai:debug` — systematyczny debugging

Root cause investigation zanim cokolwiek zostanie naprawione.

```bash
/absolut-ai:debug testy w module auth padaja na CI ale przechodza lokalnie
```

4 fazy: Root Cause → Pattern Analysis → Hypothesis → Implementation.

Eskaluje do architecture review jeśli 3+ fixy zawiodą.

### `/absolut-ai:update-ai-context` — dokumentacja AI

Bootstrapuje lub odświeża dokumentację AI dla projektu.

```bash
/absolut-ai:update-ai-context
```

- **Brak CLAUDE.md** → Bootstrap: tworzy pełną dokumentację od zera
- **CLAUDE.md istnieje** → Update: audytuje drift, odkrywa nowe wzorce

Zarządza: `CLAUDE.md`, `./absolut-ai/patterns.md`, `./absolut-ai/rules.md`

#### Kiedy uruchamiać

Warto odpalać regularnie na projektach z istniejącym kodem — np. co 1-2 tygodnie lub po większych zmianach (nowy moduł, refactor architektury, zmiana stacku).

**Co robi przy update:**
- **Drift detection** — wykrywa rozbieżności między `CLAUDE.md` a aktualnym kodem: usunięte pliki, zmienione nazwy, nowe zależności, nieaktualne ścieżki
- **Pattern discovery** — skanuje codebase pod kątem nowych wzorców, które warto udokumentować w `patterns.md` (np. nowy typ serwisu, nowy sposób obsługi błędów)
- **Rules audit** — weryfikuje czy `rules.md` nadal odzwierciedla faktyczne konwencje w kodzie

**Dlaczego to ważne:** `CLAUDE.md` który nie nadąża za kodem jest gorszy niż brak `CLAUDE.md` — agent podejmuje decyzje na podstawie nieaktualnych informacji. Regularny update zapobiega temu dryfowi.

## Konwencja plików

Wszystkie artefakty w `./absolut-ai/` w roocie projektu:

```
projekt/
├── absolut-ai/
│   ├── feature/
│   │   ├── planning-{slug}.md
│   │   └── tasks-{slug}.md
│   ├── reviews/
│   │   └── YYYY-MM-DD-{branch-slug}.md
│   ├── patterns.md
│   └── rules.md
├── docs/adr/
│   └── YYYY-MM-DD-{slug}.md
└── CLAUDE.md
```

## Typowe workflow

### Nowy feature

```bash
/absolut-ai:feature-discuss "system powiadomien push"
/absolut-ai:generate-tasks @absolut-ai/feature/planning-push-notifications.md
/absolut-ai:implement @absolut-ai/feature/tasks-push-notifications.md
/absolut-ai:review
```

### Bug fix

```bash
/absolut-ai:debug "endpoint /api/users zwraca 500 przy pustym query param"
```

### Setup projektu

```bash
/absolut-ai:update-ai-context
```

## Relacja do Lightweight Flow

Absolut AI Skills to ewolucja [Lightweight Flow](kamil-lightweight-flow/) w formie pluginu. Kluczowe różnice:

- **Dystrybucja** — instalacja jedną komendą zamiast ręcznego kopiowania plików
- **Aktualizacje** — `plugin install` zamiast ponownego kopiowania
- **Namespace** — `/absolut-ai:` zamiast luźnych `/feature`, `/tasks:`, `/nexus:`
- **Artefakty** — pliki w `./absolut-ai/` zamiast w roocie projektu (`FEATURE.md`, `TASKS.md`)
- **Debug** — nowy skill `/absolut-ai:debug` z 4-fazowym procesem
- **AI Context** — `/absolut-ai:update-ai-context` zastępuje `/tasks:ai-enable-project` i `/tasks:ai-update-project`

## Struktura repo

```
absolut-ai-skills/
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
├── skills/
│   ├── debug/
│   ├── feature-discuss/
│   ├── generate-tasks/
│   ├── implement/
│   ├── review/
│   └── update-ai-context/
├── package.json
└── README.md
```

## Wymagania

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI lub rozszerzenie IDE

## Licencja

MIT — Absolut Systems
