# Git Worktrees w Claude Code

Praktyczny przewodnik po uruchamianiu równoległych sesji Claude Code z wykorzystaniem git worktrees — dla programistów, którzy chcą pracować nad wieloma zadaniami jednocześnie bez konfliktów i utraty kontekstu.

## Treść

### Problem: jeden katalog roboczy, wiele zadań

Domyślnie repozytorium git ma tylko jedno drzewo robocze. Każda gałąź, którą sprawdzasz, żyje w tym samym zbiorze plików na dysku. Jeśli Claude w połowie generuje nową funkcję na gałęzi `feature/payments`, katalog roboczy jest w częściowo zmodyfikowanym stanie — nie możesz bezpiecznie uruchomić drugiej sesji dotykającej innych plików.

Efekt: praca szeregowa. Jedno zadanie na raz.

### Rozwiązanie: git worktrees

Zamiast jednego folderu z repozytorium, worktrees dają wiele folderów — każdy na własnej gałęzi, wszystkie wskazujące na tę samą bazę danych git. Worktrees zajmują dodatkowe miejsce na dysku, ale ponieważ współdzielą główny katalog `.git`, ogólne zużycie pozostaje niskie.

```
my-project/           ← główny worktree (main)
my-project-payments/  ← linked worktree (feature/payments)
my-project-refactor/  ← linked worktree (feature/refactor)
```

Każdy katalog jest w pełni niezależny — możesz uruchomić Claude Code we wszystkich jednocześnie.

### Uruchomienie

**Wbudowana flaga (od v2.1.50):**

```bash
# Tworzy .claude/worktrees/feature-auth/ z nową gałęzią
claude --worktree feature-auth

# Drugi, równoległy worktree
claude --worktree bugfix-123
```

**Ręcznie przez git (więcej kontroli nad lokalizacją i gałęzią):**

```bash
git worktree add ../project-feature-a -b feature-a
git worktree add ../project-bugfix bugfix-123
cd ../project-feature-a && claude
```

Dodaj `.claude/worktrees/` do `.gitignore`, żeby zawartość worktrees nie pojawiała się jako nieśledzone pliki w głównym repozytorium.

### Worktrees dla subagentów

Subagenty mogą korzystać z izolacji worktree, aby wykonywać więcej pracy równolegle — szczególnie przydatne przy dużych zmianach wsadowych i migracjach kodu.

Najprostszy sposób — powiedz Claude w sesji głównej:

```
Użyj worktrees dla subagentów.
Agent 1: napisz testy dla modułu auth
Agent 2: napisz testy dla modułu payments
Następnie scal obie gałęzie do main.
```

Możesz też dodać `isolation: worktree` w frontmatter definicji subagenta:

```yaml
---
name: isolated-implementer
description: "Implementuje zmiany w izolowanym worktree"
isolation: worktree
---
```

### Po zakończeniu pracy

- **Brak zmian** — worktree i jego gałąź są usuwane automatycznie.
- **Są zmiany** — Claude pyta, czy zachować worktree; merge i cleanup robisz ręcznie:

```bash
# Scal zmiany do main
git merge worktree-feature-auth
git merge worktree-bugfix-123

# Usuń worktrees
git worktree remove .claude/worktrees/feature-auth
git worktree remove .claude/worktrees/bugfix-123

# Opcjonalnie: wyczyść stare metadane
git worktree prune
```

Przed usunięciem worktree warto użyć `/rename` w sesji Claude, żeby nazwać ważne sesje — usunięcie worktree może spowodować, że powiązane sesje staną się niedostępne.

### Kiedy worktrees pomagają, a kiedy nie

| Sytuacja | Rekomendacja |
|---|---|
| Agenci dotykają różnych plików/modułów | ✅ Działają bez problemów |
| Agenci dotykają tych samych plików | ⚠️ Izolują pracę, ale merge wymaga uwagi |
| Zadania muszą wykonać się w kolejności | ❌ Nie używaj równoległych agentów |
| Zadania są logicznie sprzeczne | ❌ Worktrees nie pomogą |

### Kluczowa zasada

Worktrees rozwiązują problem **techniczny** — eliminują konflikty na poziomie systemu plików podczas pracy. Nie rozwiązują problemu **architektonicznego** — jeśli dwóch agentów podejmuje sprzeczne decyzje projektowe w tym samym module, konflikt i tak pojawi się przy merge.

Dlatego dobry podział zadań między agentów jest ważniejszy niż samo użycie worktrees. Worktrees to zabezpieczenie izolacji, nie sposób na pracę nad tym samym kodem w tym samym czasie.