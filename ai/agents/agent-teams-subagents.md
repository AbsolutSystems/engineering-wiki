# Subagenty i orkiestracja w Claude Code

Przewodnik po tworzeniu własnych subagentów i zarządzaniu nimi w Claude Code — dla programistów, którzy chcą rozbić złożone zadania na wyspecjalizowanych pomocników pracujących równolegle.

## Treść

### Czym jest subagent?

Subagent to wyspecjalizowana instancja Claude uruchamiana przez agenta głównego do wykonania konkretnego, ograniczonego zadania. Ma własne okno kontekstu, własny prompt systemowy i własne uprawnienia do narzędzi. Po zakończeniu zwraca wynik do agenta głównego.

Format pliku agenta i subagenta jest identyczny. Rozróżnienie jest wyłącznie behawioralne — chodzi o rolę, jaką instancja odgrywa w czasie wykonania, a nie o sposób zapisu pliku. Ten sam plik `architecture-reviewer.md` może być Twoim agentem najwyższego poziomu gdy wywołujesz go bezpośrednio, albo subagenten gdy orkiestrator uruchamia go jako jeden krok w większym pipeline'ie.

### Tworzenie subagenta

Subagent to plik Markdown z frontmatter YAML, umieszczony w `.claude/agents/` (zakres projektu) lub `~/.claude/agents/` (zakres użytkownika).

```markdown
---
name: code-reviewer
description: Ekspert od code review. Używaj po napisaniu lub zmodyfikowaniu kodu.
tools: Read, Grep, Glob, Bash
model: claude-sonnet-4-6
permissionMode: default
---

Jesteś seniorem code reviewerem. Sprawdzaj:
- czytelność i nazewnictwo
- obsługę błędów i edge case'ów
- brak sekretów w kodzie
- pokrycie testami
```

Subagenty mogą działać na pierwszym planie (blokująco, pytania o uprawnienia przekazywane do użytkownika) lub w tle (współbieżnie, uprawnienia pre-zatwierdzone).

Kluczowe pola frontmatter:

- `name` — identyfikator, po którym Claude odnajduje agenta
- `description` — opis kiedy go używać; Claude dobiera agenta na podstawie tego pola
- `tools` — lista dozwolonych narzędzi (ograniczenie = lepsza jakość)
- `model` — można użyć tańszego modelu dla prostszych zadań
- `isolation: worktree` — opcjonalne, uruchamia agenta w osobnym worktree

### Uruchamianie subagentów

Najprostszy sposób to polecenie w języku naturalnym w sesji głównej:

```
Użyj subagenta code-reviewer, żeby sprawdzić ostatnie zmiany w src/auth.ts
```

Dla zadań równoległych:

```
Uruchom równolegle:
- subagent backend-architect: zaproponuj strukturę API dla modułu płatności
- subagent test-writer: napisz testy jednostkowe dla modułu auth
```

Automatyczne wywołanie bywa zawodne — Claude może obsłużyć zadanie samodzielnie zamiast delegować do subagenta. Bezpieczniej jest jawnie wskazywać: "użyj subagenta code-reviewer" zamiast liczyć na autodetekcję.

### Subagenty vs Agent Teams

Wybór zależy od tego, czy pracownicy muszą komunikować się ze sobą: subagenty stosuj gdy potrzebujesz szybkich, skupionych pracowników raportujących wyniki. Agent Teams stosuj gdy potrzebujesz współpracy, dzielenia się wynikami i wzajemnej koordynacji.

| | Subagenty | Agent Teams |
|---|---|---|
| Komunikacja | tylko z agentem głównym | między sobą nawzajem |
| Koordynacja | przez agenta głównego | wspólna lista zadań |
| Koszt tokenów | niski | ~3–4x więcej |
| Kiedy używać | izolowane, równoległe zadania | złożona współpraca między domenami |

### Agent Teams — włączenie i użycie

Agent Teams są domyślnie wyłączone. Aby je włączyć, dodaj do `settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Następnie opisz zespół w języku naturalnym:

```
Stwórz agent team z 3 osobami:
- architekt backendu: projektuje API i schemat bazy
- developer frontendu: implementuje UI
- tester: pisze testy i weryfikuje integrację
```

Możesz przejść do dowolnego członka zespołu i dać mu dodatkowe instrukcje, zadać pytania lub zmienić kierunek pracy w trakcie wykonania — tak jakbyś otworzył kolejną instancję Claude Code.

### Kiedy nie używać równoległych agentów

Agent Teams dodają narzut koordynacyjny i zużywają znacznie więcej tokenów niż pojedyncza sesja. Działają najlepiej gdy członkowie mogą pracować niezależnie. Dla zadań sekwencyjnych, edycji tych samych plików lub pracy z wieloma zależnościami — lepsza jest pojedyncza sesja lub subagenty.

### Dobre praktyki

Specjalizacja bije generalizację. Subagent z wąskim promptem i ograniczonym zestawem narzędzi daje lepsze wyniki niż ogólna sesja z dostępem do wszystkiego. Ograniczenie jest cechą, nie wadą.

- Pisz precyzyjne `description` — Claude dobiera agenta właśnie na tej podstawie
- Ograniczaj `tools` do minimum potrzebnego dla danej roli
- Używaj tańszych modeli (Haiku, Sonnet) dla prostszych zadań, droższych (Opus) dla złożonych
- Dla zadań dotykających tych samych plików dodaj `isolation: worktree`