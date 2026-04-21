# Engineering Wiki

Wewnętrzna baza wiedzy dla zespołu developerskiego. Znajdziesz tu artykuły, poradniki i dobre praktyki dotyczące naszego stacku technologicznego, procesów i narzędzi.

## Struktura

Artykuły są pogrupowane tematycznie w katalogach:

```
engineering-wiki/
├── README.md
├── backend/
├── frontend/
├── devops/
├── ai/
├── architecture/
└── processes/
```

## Jak dodać artykuł

1. Wybierz odpowiedni katalog tematyczny (lub utwórz nowy, jeśli żaden nie pasuje).
2. Utwórz plik Markdown (`.md`) z opisową nazwą, np. `docker-best-practices.md`.
3. Zacznij artykuł od nagłówka H1 i krótkiego opisu:

```markdown
# Tytuł artykułu

Krótki opis — o czym jest artykuł i dla kogo jest przeznaczony.

## Treść

...
```

4. Stwórz pull request z opisem zmian.

## Konwencje

- **Język plików** — artykuły piszemy po polsku.
- **Nazwy plików** — kebab-case, np `wersjonowanie-api.md`.
- **Obrazki** — umieszczaj w katalogu `assets/` obok artykułu i linkuj relatywnie.
- **Formatowanie** — używaj standardowego Markdowna (nagłówki, listy, bloki kodu z oznaczeniem języka).

## Spis treści

### AI

- [Get Shit Done (GSD)](ai/get-shit-done.md) — spec-driven development z Claude Code i innymi AI coding agents
- [Lightweight Flow](ai/kamil-lightweight-flow/) — workflow Kamila z Claude Code: `/feature` + `/tasks` + `/nexus`
- [Dokumentacja projektu dla AI (CLAUDE.md)](ai/claude-md-documentation/) — system hierarchicznych plików CLAUDE.md dla AI coding agents
- [Superpowers](ai/superpowers.md) — composable skills wymuszające ustrukturyzowany workflow (TDD, planning, code review)
- [Absolut AI Skills](ai/absolut-ai-skills.md) — plugin do Claude Code: feature-discuss → generate-tasks → implement → review
