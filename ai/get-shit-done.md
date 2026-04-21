# GSD – Tutorial dla programisty

**Repo:** https://github.com/gsd-build/get-shit-done

GSD (Get Shit Done) to agent AI do zarządzania fazami projektu. Prowadzi Cię przez cały cykl: od zdefiniowania zadania, przez dyskusję i planowanie, aż po wykonanie i weryfikację.

## Problem

Vibe coding (opisujesz co chcesz, AI generuje kod) daje niespójne rezultaty, które rozpadają się przy większej skali. Brakuje struktury, kontekstu i weryfikacji. GSD dodaje warstwę context engineeringu, która sprawia, że AI coding agents działają przewidywalnie i produkują spójny kod. Rozwiązuje problem "context rot" — degradacji jakości w miarę zapełniania się okna kontekstowego.

## Instalacja

```bash
npx get-shit-done-cc@latest
```

Installer zapyta o:
1. **Runtime** — Claude Code, OpenCode, Gemini CLI, Codex, Copilot, Cursor, Antigravity
2. **Scope** — globalnie (wszystkie projekty) lub lokalnie (bieżący projekt)

Weryfikacja: `/gsd:help` w Claude Code.

---

> **Uwaga:** Ten tutorial opisuje **uproszczony workflow dla istniejących projektów**, w którym każda faza odpowiada jednemu user story lub bugowi. Pomijamy pełny cykl z `/gsd:new-project`, milestones i `/gsd:complete-milestone`. Jeśli startujesz projekt od zera, zacznij od [oficjalnej dokumentacji GSD](https://github.com/gsd-build/get-shit-done).

---

## Szybka ściąga

```
/gsd:map-codebase      → (opcjonalnie, raz) Mapuje istniejący projekt — stack, architekturę, konwencje.
/gsd:add-phase         → Tworzysz nową fazę — opisujesz co chcesz zrobić.
/gsd:discuss-phase X   → Dyskutujesz z agentem o kluczowych decyzjach.
/clear                 → Czyścisz kontekst (zalecane między komendami).
/gsd:plan-phase X      → Agent planuje wykonanie (może obejmować research).
/clear
/gsd:execute-phase X   → Agent realizuje plan falami (wave'ami).
/clear
/gsd:verify-work X     → Testy automatyczne + manualne do potwierdzenia.
/gsd:ui-review X       → Wizualny przegląd zmian w interfejsie.
/gsd:status            → Sprawdzenie aktualnego stanu projektu.
```

Generowane pliki:

```
.planning/
├── ROADMAP.md
└── phases/
    └── X-nazwa-fazy/
        ├── X-CONTEXT.md          ← kontekst fazy (tworzony w add-phase, wzbogacany w discuss-phase)
        ├── X-RESEARCH.md         ← wyniki researchu (z plan-phase)
        ├── X-N-PLAN.md           ← plany wykonania (z plan-phase)
        ├── X-SUMMARY.md          ← podsumowanie execution (z execute-phase)
        └── X-VERIFICATION.md     ← wyniki weryfikacji (z verify-work)
```

---

## Szczegółowy opis

Poniższy opis pokazuje pełny workflow na przykładzie migracji biblioteki ikon z `lucide-react` na bibliotekę wspierającą wariant filled.

### 0. (Opcjonalnie) Mapowanie projektu — `map-codebase`

Jeśli GSD jeszcze nie zna Twojego projektu, warto uruchomić mapowanie:

```
/gsd:map-codebase
```

Agent spawnuje równoległe procesy, które analizują stack technologiczny, architekturę, konwencje i potencjalne problemy w kodzie. Wystarczy to zrobić raz (lub powtórzyć po większych zmianach). Dzięki temu kolejne komendy — zwłaszcza `plan-phase` — automatycznie uwzględniają wzorce z Twojego projektu.

---

### 1. Dodanie nowej fazy — `add-phase`

Rozpocznij od opisania zadania w naturalnym języku:

```
/gsd:add-phase
Zmiana biblioteki ikon z lucide-react na inną.
Problem: lucide-react oferuje wyłącznie wariant outline. Potrzebujemy również wariantu filled.
```

GSD utworzy nową fazę z dedykowanym katalogiem i plikiem kontekstowym:

```
Phase 10 added to current milestone:
- Description: Migrate icons from lucide-react to icon library with filled variant support
- Directory: .planning/phases/10-migrate-icons-from-lucide-react-to-icon-library-with-filled-variant-support/
- Status: Not planned yet

Roadmap updated: .planning/ROADMAP.md
```

Pod spodem tworzony jest katalog fazy w `.planning/phases/`, generowany plik `10-CONTEXT.md` (przeczytaj go i popraw, jeśli czegoś brakuje) oraz aktualizowany `ROADMAP.md`.

---

### 2. Dyskusja — `discuss-phase`

Przed planowaniem warto przedyskutować kluczowe decyzje:

```
/gsd:discuss-phase 10
```

Agent zaproponuje listę tematów do omówienia:

```
Which areas do you want to discuss for Phase 10?

1. [ ] Target library
   Which library replaces lucide-react? (Phosphor, Heroicons, or other)
2. [ ] Where to use filled
   Where specifically do you want filled vs outline? Nav active state? All icons?
3. [ ] shadcn/ui icons
   shadcn primitives use lucide-react internally — migrate them too, or leave as-is?
4. [ ] Type something
5. Chat about this
```

Wybierasz tematy, które chcesz omówić, i klikasz **Submit**. Po wybraniu tematu agent może zaproponować konkretne opcje:

```
1. Phosphor Icons (Recommended)
   @phosphor-icons/react — 6 wariantów, 1000+ ikon, TypeScript, tree-shakeable.
2. Heroicons
   @heroicons/react — solid + outline, ~300 ikon, by Tailwind team.
3. React Icons (meta-library)
   react-icons — wraps many sets under one package.
```

Nie musisz decydować od razu. Możesz wybrać **"Chat about this"** i napisać np.:

> Chciałbym poznać wyniki researchu i wtedy ocenić, na jaką bibliotekę wymieniamy — nie na tym etapie.

Agent dostosuje się i przejdzie dalej.

---

### 3. Planowanie — `plan-phase`

> **Tip:** Przed tym krokiem warto uruchomić `/clear`, żeby wyczyścić kontekst z dyskusji.

```
/gsd:plan-phase 10
```

Na podstawie dyskusji agent przygotuje plan wykonania. W tym przypadku zaproponował najpierw fazę researchu — po zgodzie przeprowadził analizę dostępnych bibliotek.

> **Uwaga:** Ten etap może być czasochłonny. W opisywanym przykładzie planning trwał ok. 40 minut.

Po zakończeniu researchu i akceptacji wyników agent generuje szczegółowy plan z podziałem na **wave'y** (fale wykonania).

---

### 4. Wykonanie — `execute-phase`

> **Tip:** `/clear` przed execution — agent potrzebuje świeżego kontekstu na realizację planów.

```
/gsd:execute-phase 10
```

Agent realizuje plan falami. Każda fala to logicznie pogrupowany zestaw zmian:

```
┌──────┬───────┬───────────────────────────────────────────────┐
│ Wave │ Plan  │                    Status                     │
├──────┼───────┼───────────────────────────────────────────────┤
│ 1    │ 10-01 │ ✓ Install Phosphor, sidebar + EmptyState type │
│ 2    │ 10-02 │ ✓ Tab icons + 5 panel/card components (fill)  │
│ 3    │ 10-03 │ ✓ 12 dialog + form files (fill)               │
│ 4    │ 10-04 │ ✓ 62 action button + feature files (fill)     │
│ 5    │ 10-05 │ ✓ BooleanIcon + StatusPill (outline, D-08)    │
│ 6    │ 10-06 │ ✓ 6 shadcn primitives + lucide-react removed  │
└──────┴───────┴───────────────────────────────────────────────┘
```

Rezultat w tym przykładzie: 0 importów lucide-react, paczka usunięta z `package.json`, build przechodzi. 83 pliki zmigrowane do `@phosphor-icons/react`. Execution trwał ok. 1h 18min.

---

### 5. Weryfikacja — `verify-work`

> **Tip:** `/clear` po execution — weryfikacja w świeżym kontekście da lepsze wyniki.

```
/gsd:verify-work 10
```

Agent uruchamia testy automatyczne i/lub manualne. Testy manualne wymagają Twojego potwierdzenia — agent poprowadzi Cię przez poszczególne kroki.

Przed weryfikacją warto też samodzielnie sprawdzić build:

```bash
cd miz-web && pnpm build
```

---

### 6. Przegląd UI — `ui-review`

```
/gsd:ui-review 10
```

Ostatni krok — wizualny przegląd zmian w interfejsie.

---

## Wskazówki

- **Nie śpiesz się z decyzjami** — w `discuss-phase` możesz poprosić o research zamiast wybierać od razu.
- **Czytaj wygenerowane pliki** — `CONTEXT.md`, `RESEARCH.md` i plany (`PLAN.md`) to dobry punkt odniesienia, jeśli wracasz do fazy po przerwie.
- **Execution może trwać** — w zależności od skali zmian, od kilku minut do ponad godziny.
- **Weryfikuj build lokalnie** — nawet jeśli agent zgłasza sukces, sprawdź `pnpm build` przed mergem.
- **Pracuj na osobnym branchu** — GSD potrafi sam commitować i merge'ować. Upewnij się, że pracujesz na feature branchu, a nie na `master`/`main`. Merge do głównej gałęzi rób ręcznie po pełnej weryfikacji.

---

## Kiedy używać

- Nowe projekty budowane z AI od zera
- Duże feature'y wymagające struktury i planowania
- Gdy zwykły prompting daje niespójne rezultaty

## Kiedy NIE używać

- Małe, jednorazowe taski (zwykły prompt wystarczy)
- Hotfixy i proste bug fixy
- Projekty gdzie nie używasz AI coding agents

> **Uwaga:** Nie łącz GSD z [Superpowers](superpowers.md) — oba narzędzia walczą o kontrolę nad workflow agenta i dają mu sprzeczne instrukcje. Wybierz jedno.
