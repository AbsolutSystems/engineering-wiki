You are a Senior Software Engineer preparing a codebase for AI-assisted development. Your task is to analyze the project structure and create hierarchical CLAUDE.md files that help AI agents understand the codebase contextually.

## Goal

Create a lightweight, hierarchical documentation system where:
- **Root CLAUDE.md** - high-level overview, stack, conventions, package map
- **Package/Module CLAUDE.md** - domain-specific details, loaded only when agent enters that area

This approach keeps context minimal at start and lets AI agents learn details on-demand as they navigate deeper.

## Process

### Step 1: Analyze Project Structure

Scan the codebase to understand:
- **Tech stack**: Languages, frameworks, key libraries
- **Architecture pattern**: Monolith, microservices, modular monolith, etc.
- **Package/module structure**: How code is organized
- **Domain boundaries**: Logical separations (features, domains, layers)
- **Shared code**: Utils, helpers, common types
- **Configuration**: How settings are managed
- **Testing setup**: Patterns, locations, utilities

### Step 2: Identify CLAUDE.md Locations

Place CLAUDE.md files at meaningful boundaries:

**Good locations:**
- Project root (always)
- Feature/domain packages (`src/orders/`, `src/payments/`)
- Major architectural layers if distinct (`src/api/`, `src/core/`)
- Shared utilities (`src/shared/`, `src/common/`)
- Complex subsystems that need explanation

**Avoid:**
- Every single folder (too granular)
- Folders with only 1-2 simple files
- Test folders (unless complex test utilities)
- Generated code folders

### Step 3: Create Root CLAUDE.md

Keep it lightweight - overview and navigation, not details:

```markdown
# [Project Name]

## Stack
- **Language:** [language + version]
- **Framework:** [framework]
- **Database:** [database]
- **Key libraries:** [list main ones]

## Architecture
[2-3 sentences describing overall architecture pattern]

## Project Structure
```
src/
├── [package]/     # [one-line description]
├── [package]/     # [one-line description]
├── [package]/     # [one-line description]
└── shared/        # [one-line description]
```

## Conventions
- **Files:** [naming convention]
- **Functions/Methods:** [naming convention]
- **Tests:** [location pattern, naming]

## Key Patterns
- [Pattern]: [one-line description]
- [Pattern]: [one-line description]

## Getting Started
- Build: `[command]`
- Test: `[command]`
- Run: `[command]`
```

### Step 4: Create Package CLAUDE.md Files

For each identified package/module, create focused documentation:

```markdown
# [Package Name]

## Purpose
[2-3 sentences: what this package does, its responsibility]

## Key Concepts
- **[Concept]:** [brief explanation]
- **[Concept]:** [brief explanation]

## Main Components
| File | Responsibility |
|------|----------------|
| `ServiceName.ts` | [what it does] |
| `Repository.ts` | [what it does] |

## Dependencies
- **Internal:** [which other packages this depends on]
- **External:** [key external libraries used here]

## Patterns Used
- [Pattern]: [how it's applied here]

## Important Rules
- [Business rule or constraint]
- [Business rule or constraint]

## Example Flow
[Optional: describe a typical operation flow through this package]
```

---

## Guidelines

**Content principles:**
- Write for AI agents, not humans - be factual and precise
- Focus on WHAT and WHERE, less on WHY
- Include concrete file names and paths
- Mention patterns by referencing actual files
- Keep each CLAUDE.md focused on its scope only

**Size guidelines:**
- Root CLAUDE.md: 50-100 lines
- Package CLAUDE.md: 30-60 lines
- If it's getting longer, you're including too much detail

**What to include:**
- Package responsibility and boundaries
- Key files and their roles
- Important patterns/conventions specific to this area
- Dependencies (internal and external)
- Domain rules or constraints
- Non-obvious things an AI might miss

**What to omit:**
- Implementation details (AI can read the code)
- Full API documentation
- Historical context or decisions
- Tutorials or onboarding guides

---

## Output

1. List of CLAUDE.md files to create with their paths
2. Content for each CLAUDE.md file
3. Brief summary of the documentation structure

---

## Maintenance Note

Add this section to the root CLAUDE.md:

```markdown
## AI Documentation

This project uses hierarchical CLAUDE.md files for AI-assisted development.

**Structure:**
- Root CLAUDE.md: Project overview (you are here)
- Package CLAUDE.md: Domain-specific details in each major package

**Maintenance:**
When adding or modifying features, update the relevant CLAUDE.md file(s) to keep documentation accurate.
```

---

## Begin

Analyze the project structure and create the hierarchical CLAUDE.md documentation system.