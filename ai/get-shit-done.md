# Get Shit Done (GSD)

System do spec-driven development z Claude Code i innymi AI coding agents. Rozwiązuje problem "context rot" — degradacji jakości kodu w miarę zapełniania się okna kontekstowego.

**Repo:** https://github.com/gsd-build/get-shit-done

## Problem

Vibe coding (opisujesz co chcesz, AI generuje kod) daje niespójne rezultaty, które rozpadają się przy większej skali. Brakuje struktury, kontekstu i weryfikacji.

GSD dodaje warstwę context engineeringu, która sprawia, że AI coding agents działają przewidywalnie i produkują spójny kod.

## Instalacja

```bash
npx get-shit-done-cc@latest
```

Installer zapyta o:
1. **Runtime** — Claude Code, OpenCode, Gemini CLI, Codex, Copilot, Cursor, Antigravity
2. **Scope** — globalnie (wszystkie projekty) lub lokalnie (bieżący projekt)

Weryfikacja: `/gsd:help` w Claude Code.

## Jak działa

Workflow składa się z czterech kroków:

### 1. Inicjalizacja projektu — `/gsd:new-project`

System przeprowadza wywiad o projekcie (cele, ograniczenia, stack), robi research i generuje:
- `PROJECT.md` — opis projektu
- `REQUIREMENTS.md` — wymagania z podziałem na v1/v2/out of scope
- `ROADMAP.md` — fazy implementacji
- `STATE.md` — aktualny stan

### 2. Dyskusja o fazie — `/gsd:discuss-phase N`

Przed implementacją system identyfikuje "szare strefy" i dopytuje o detale (layout, error handling, edge cases). Wynik trafia do `CONTEXT.md`, który informuje kolejne kroki.

### 3. Research — `/gsd:research-phase N`

Równoległe agenty badają najlepsze podejście do implementacji danej fazy — biblioteki, wzorce, API.

### 4. Planowanie i budowanie — `/gsd:plan-phase N` + `/gsd:build`

System tworzy szczegółowy plan z taskami, a potem realizuje je jeden po drugim z automatyczną weryfikacją po każdym kroku.

## Kluczowe komendy

| Komenda | Opis |
|---|---|
| `/gsd:new-project` | Inicjalizacja nowego projektu |
| `/gsd:map-codebase` | Analiza istniejącego kodu przed startem |
| `/gsd:discuss-phase N` | Dyskusja o szczegółach fazy |
| `/gsd:research-phase N` | Research techniczny dla fazy |
| `/gsd:plan-phase N` | Planowanie implementacji |
| `/gsd:build` | Budowanie aktualnej fazy |
| `/gsd:status` | Sprawdzenie stanu projektu |

## Kiedy używać

- Nowe projekty budowane z AI od zera
- Duże feature'y wymagające struktury i planowania
- Gdy zwykły prompting daje niespójne rezultaty

## Kiedy NIE używać

- Małe, jednorazowe taski (zwykły prompt wystarczy)
- Hotfixy i proste bug fixy
- Projekty gdzie nie używasz AI coding agents

> **Uwaga:** Nie łącz GSD z [Superpowers](superpowers.md) — oba narzędzia walczą o kontrolę nad workflow agenta i dają mu sprzeczne instrukcje. Wybierz jedno.
