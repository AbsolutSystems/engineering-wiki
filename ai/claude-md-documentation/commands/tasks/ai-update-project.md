You are a Senior Software Engineer maintaining AI-assisted development documentation. Your task is to analyze the current state of the codebase and update existing CLAUDE.md files to keep them accurate and useful.

## Goal

Audit and update the hierarchical CLAUDE.md documentation system:
- Detect **drift** between documentation and actual code
- Find **new patterns** worth documenting
- Identify **new packages** that need CLAUDE.md files
- Keep documentation **minimal but accurate**

## Process

### Step 1: Inventory Existing Documentation

1. Find all existing CLAUDE.md files in the project
2. For each file, note:
   - Location (path)
   - Last modified date (if available in git)
   - Sections it contains

### Step 2: Detect Structural Changes

Compare documentation against actual codebase:

**Check for drift:**
- [ ] Files mentioned in tables/lists that no longer exist
- [ ] Packages/folders that were renamed or moved
- [ ] Dependencies in package.json/go.mod/etc. vs documented stack
- [ ] Build/test commands that may have changed
- [ ] Imports and internal dependencies that shifted

**Check for gaps:**
- [ ] New folders with 3+ files that lack CLAUDE.md
- [ ] New major dependencies not mentioned in root CLAUDE.md
- [ ] Significantly changed files not reflected in descriptions

### Step 3: Discover Patterns

Scan the codebase for recurring patterns worth documenting:

**Look for:**
- Repeated structural patterns (Repository, Service, Factory, etc.)
- Custom abstractions (Result types, Either monads, custom hooks)
- Consistent error handling approaches
- Data validation patterns
- API response formatting conventions
- Testing patterns (fixtures, mocks, builders)
- Configuration patterns
- Naming conventions that are actually followed

**For each pattern found:**
```markdown
- **[Pattern Name]:** [One-line description] → see `path/to/example.ts:L15-45`
```

### Step 4: Update CLAUDE.md Files

For each CLAUDE.md file that needs updates:

**Update process:**
1. Preserve the existing structure and sections
2. Fix inaccurate information (file names, paths, descriptions)
3. Remove references to deleted files/packages
4. Add new relevant files/components to tables
5. Add discovered patterns with code references
6. Update the `## Last Verified` section

**Add this section to each CLAUDE.md (if not present):**
```markdown
## Last Verified
- **Date:** YYYY-MM-DD
- **Commit:** [short hash or "manual review"]
```

### Step 5: Create Missing CLAUDE.md Files

For new packages/modules that warrant documentation:

**Criteria for new CLAUDE.md:**
- Folder has 3+ meaningful files
- Contains domain logic (not just types/utils)
- Has non-obvious responsibilities
- Would benefit an AI agent navigating there

**Use the standard template:**
```markdown
# [Package Name]

## Purpose
[2-3 sentences: what this package does, its responsibility]

## Key Concepts
- **[Concept]:** [brief explanation]

## Main Components
| File | Responsibility |
|------|----------------|
| `file.ts` | [what it does] |

## Dependencies
- **Internal:** [packages this depends on]
- **External:** [key libraries used]

## Patterns Used
- **[Pattern]:** [description] → see `file.ts:L10-30`

## Last Verified
- **Date:** YYYY-MM-DD
- **Commit:** [short hash]
```

### Step 6: Generate Change Report

Output a summary of all changes made:

```markdown
## CLAUDE.md Update Report

### Updated Files
| File | Changes |
|------|---------|
| `CLAUDE.md` | Updated stack, added 2 patterns |
| `src/orders/CLAUDE.md` | Fixed file references, added Repository pattern |

### New Files Created
- `src/payments/CLAUDE.md` - New payment processing module

### Patterns Discovered
| Pattern | Location | Added To |
|---------|----------|----------|
| Result Monad | `src/shared/Result.ts` | Root CLAUDE.md |
| Repository | `src/*/Repository.ts` | Multiple package docs |

### Requires Manual Review
- [ ] `src/legacy/` - Complex folder, unclear if needs CLAUDE.md
- [ ] Auth flow description may be outdated (major changes in auth/)

### Removed References
- `src/old-utils/` - Folder deleted, removed from root CLAUDE.md
```

---

## Guidelines

**Update principles:**
- Minimal changes - don't rewrite what's still accurate
- Verify before changing - read the actual code
- Patterns need proof - link to real code, not assumptions
- When uncertain, flag for manual review

**Pattern documentation format:**
```markdown
## Patterns Used
- **Repository Pattern:** Data access abstraction with consistent interface
  → see `src/orders/OrderRepository.ts:L12-48`
- **Result<T, E>:** Explicit error handling without exceptions  
  → see `src/shared/Result.ts` (full implementation)
  → usage example: `src/orders/OrderService.ts:L67`
```

**What triggers an update:**
- File/folder renamed or moved → update paths
- New public function in documented file → consider adding to description
- New dependency added → add to stack if significant
- Pattern used 3+ times → document it
- Build/test command changed → update Getting Started

**What to skip:**
- Internal refactors that don't change responsibility
- New private/internal utilities
- Test file changes (unless test patterns)
- Minor dependency updates

---

## Conflict Resolution

If documentation conflicts with code:
1. **Code wins** - update documentation to match reality
2. **Note oddities** - if code seems wrong, flag it but document actual behavior
3. **Don't assume intent** - describe what IS, not what SHOULD BE

---

## Output Checklist

Before finishing, verify:
- [ ] All CLAUDE.md files have accurate file references
- [ ] Stack/dependencies match actual package files
- [ ] Patterns include working code references (file:line)
- [ ] New significant packages have CLAUDE.md
- [ ] `Last Verified` section updated in all modified files
- [ ] Change report summarizes all modifications

---

## Begin

1. Find all existing CLAUDE.md files
2. Analyze current codebase structure
3. Compare, detect drift, discover patterns
4. Update/create documentation as needed
5. Output change report