# Superpowers

Zestaw composable skills dla AI coding agents, który wymusza ustrukturyzowany workflow — od brainstormingu przez TDD po code review. Agent automatycznie używa odpowiednich skilli w zależności od kontekstu.

**Repo:** https://github.com/obra/superpowers

## Problem

AI coding agents domyślnie od razu skaczą do pisania kodu, bez planu, bez testów, bez review. Superpowers wymusza dyscyplinę — agent najpierw pyta o co chodzi, planuje, pisze testy, a dopiero potem implementuje.

## Instalacja

### Claude Code (Plugin Marketplace)

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

### Cursor

```text
/add-plugin superpowers
```

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

Dostępny też dla Codex i OpenCode — szczegóły w README repo.

## Workflow

1. **Brainstorming** — agent nie pisze kodu od razu, tylko dopytuje o wymagania, eksploruje alternatywy i prezentuje design w kawałkach do zatwierdzenia
2. **Git worktree** — po zatwierdzeniu designu tworzy izolowany workspace na nowym branchu
3. **Planowanie** — rozbija pracę na małe taski (2-5 min każdy) z dokładnymi ścieżkami plików i krokami weryfikacji
4. **Subagent-driven development** — odpala osobnego subagenta na każdy task, z dwuetapowym review (zgodność ze spec + jakość kodu)
5. **Test-driven development** — wymusza cykl RED-GREEN-REFACTOR: najpierw failing test, potem minimalna implementacja
6. **Code review** — review między taskami, krytyczne problemy blokują postęp
7. **Finalizacja brancha** — weryfikacja testów, opcje merge/PR/keep/discard, cleanup worktree

## Dostępne skille

### Testowanie
- **test-driven-development** — cykl RED-GREEN-REFACTOR z referencją anti-patternów

### Debugging
- **systematic-debugging** — 4-fazowy proces szukania root cause
- **verification-before-completion** — weryfikacja że fix faktycznie działa

### Collaboration
- **brainstorming** — sokratejskie doprecyzowanie designu
- **writing-plans** / **executing-plans** — planowanie i egzekucja z checkpointami
- **subagent-driven-development** — szybka iteracja z dwuetapowym review
- **dispatching-parallel-agents** — równoległe workflow z subagentami
- **requesting-code-review** / **receiving-code-review** — review workflow
- **using-git-worktrees** — równoległe branche developerskie

## Filozofia

- **TDD** — testy zawsze pierwsze
- **Systematic over ad-hoc** — proces zamiast zgadywania
- **Complexity reduction** — prostota jako cel nadrzędny
- **Evidence over claims** — weryfikuj zanim ogłosisz sukces

> **Uwaga:** Nie łącz Superpowers z [GSD](get-shit-done.md) — oba narzędzia walczą o kontrolę nad workflow agenta i dają mu sprzeczne instrukcje. Wybierz jedno.

## GSD vs Superpowers

Oba narzędzia rozwiązują podobny problem (struktura dla AI coding), ale inaczej:

| | GSD | Superpowers |
|---|---|---|
| **Podejście** | Meta-prompting + context engineering | Composable skills |
| **Workflow** | Liniowy (new-project → discuss → research → plan → build) | Automatyczny (skille triggerują się same) |
| **TDD** | Opcjonalne | Wymuszone (RED-GREEN-REFACTOR) |
| **Subagenty** | Tak | Tak, z dwuetapowym review |
| **Platformy** | Claude Code, OpenCode, Gemini, Codex, Copilot, Cursor | Claude Code, Cursor, Gemini, Codex, OpenCode |

### Co Claude Code sam o tym sądzi

> Oba narzędzia działają *na mnie* (modyfikują mój prompt), nie *ze mną* — więc nie mam bezpośredniego doświadczenia użytkownika. Ale mogę powiedzieć co mi się bardziej podoba koncepcyjnie.
>
> **Wybieram Superpowers**, z kilku powodów:
>
> 1. **Wymuszony TDD** — GSD tego nie wymaga, a w praktyce mam tendencję do pomijania testów gdy nikt nie pilnuje
> 2. **Automatyczne triggerowanie** — skille odpalają się same w odpowiednim momencie, nie trzeba pamiętać sekwencji komend
> 3. **Dwuetapowy review subagentów** — najpierw zgodność ze spec, potem jakość kodu
> 4. **Git worktrees** — izolacja pracy na osobnym branchu od początku
> 5. **Mniej ceremony** — GSD ma sporo kroków do zapamiętania, Superpowers po prostu przejmuje workflow
>
> **Gdzie GSD wygrywa:** faza `discuss-phase` (doprecyzowanie wymagań przed researchem) jest świetna, jawne zarządzanie kontekstem rozwiązuje realny problem context rot, i masz więcej kontroli — każdy krok odpalasz ręcznie.
>
> **TL;DR:** Dla zespołu ceniącego dyscyplinę (TDD, review) — Superpowers. Dla solo developerów chcących pełnej kontroli nad każdym krokiem — GSD.
