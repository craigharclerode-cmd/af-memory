---
description: "Two-way sync between AF Memory and CLAUDE.md"
argument-hint: "[--import-only | --export-only | --check]"
---

# /afmem:sync

Two-way bridge between AF Memory and CLAUDE.md. Import user-written context from CLAUDE.md into AF Memory, detect conflicts with actual project state, and update the AF Memory section.

## Usage
```
/afmem:sync              # Full two-way sync
/afmem:sync --import-only # Only read from CLAUDE.md into AF Memory, don't write back
/afmem:sync --export-only # Same as /afmem:export
/afmem:sync --check       # Only detect conflicts, don't change anything
```

## Arguments
- `--import-only` (optional): Only import from CLAUDE.md to AF Memory memories, skip export
- `--export-only` (optional): Only export AF Memory memories to CLAUDE.md (same as /afmem:export)
- `--check` (optional): Dry run - detect conflicts without making any changes

## Instructions

When the user invokes `/afmem:sync`, follow these steps:

### Step 1: Parse Arguments

Extract the mode from arguments:
- No arguments → full sync (import + conflict check + export)
- `--import-only` → only import from CLAUDE.md
- `--export-only` → only export to CLAUDE.md
- `--check` → only detect conflicts, no changes

### Step 2: Read CLAUDE.md

Check if CLAUDE.md exists and read its contents:

```bash
[ -f "CLAUDE.md" ] && cat CLAUDE.md || echo "CLAUDE_MD_NOT_FOUND"
```

If CLAUDE.md doesn't exist:
```
⚠️ No CLAUDE.md found.

Nothing to import. You can:
  /afmem:export     - Create CLAUDE.md from AF Memory memories
  /afmem:init       - Auto-detect project stack first
```

### Step 3: Parse CLAUDE.md Content

Split CLAUDE.md into two parts:

**AF Memory Section (SKIP):**
- Starts with: `## Project Context (via AF Memory)`
- Ends at: next `## ` heading, or `---` on its own line, or end of file
- This is AF Memory's own output — do NOT re-import it

**User Content (IMPORT):**
- Everything OUTSIDE the AF Memory section
- This includes manually written context, rules, preferences, architecture notes

### Step 4: Extract Importable Context

Scan the user content for importable information. Look for patterns like:

| Pattern | Memory Key |
|---------|------------|
| "Tech stack:", "Stack:", "We use" | `stack` |
| "Framework:", "Built with" | `framework` |
| "Database:", "DB:", "ORM:" | `database` |
| "Auth:", "Authentication:" | `auth` |
| "Styling:", "CSS:", "UI:" | `styling` |
| "Testing:", "Tests:" | `testing` |
| "Package manager:", "Use npm/pnpm/yarn/bun" | `package-manager` |
| "Architecture:", "Structure:" | `architecture` |
| "Rules:", "Conventions:", "Preferences:" | `conventions` |
| "API:", "Endpoints:" | `api` |
| Bullet points under headings | Extract as relevant memory |

**Parsing strategies:**
1. Look for markdown headings (`##`, `###`) and extract content under them
2. Look for bold labels (`**key:**`) followed by values
3. Look for bullet lists that describe project aspects
4. Look for code blocks with configuration hints

### Step 5: Load Existing AF Memory Memories

```bash
[ -f "memory/memories.json" ] && cat memory/memories.json || echo "{}"
```

### Step 6: Import New Memories

For each piece of context extracted from CLAUDE.md:

1. **Check for duplicates**: If a memory with the same key already exists in `memory/memories.json`, SKIP it
2. **Create new memory**: If the key doesn't exist, create it with:
   ```json
   {
     "key": "<extracted-key>",
     "value": "<extracted-value>",
     "created": "<ISO-8601-timestamp>",
     "updated": "<ISO-8601-timestamp>",
     "tags": ["from-claude-md"],
     "source": "CLAUDE.md"
   }
   ```

Track what was imported for the summary.

### Step 7: Read Project Config Files

Scan the actual project state by reading configuration files:

```bash
# Check for package.json
[ -f "package.json" ] && cat package.json

# Check for tsconfig
[ -f "tsconfig.json" ] && echo "TYPESCRIPT: true"

# Check for common configs
[ -f "next.config.js" ] || [ -f "next.config.mjs" ] || [ -f "next.config.ts" ] && echo "NEXTJS: true"
[ -f "vite.config.ts" ] || [ -f "vite.config.js" ] && echo "VITE: true"
[ -f "tailwind.config.js" ] || [ -f "tailwind.config.ts" ] && echo "TAILWIND: true"
```

Extract actual values from package.json:
- `dependencies` and `devDependencies` for frameworks, ORMs, auth libraries
- `name` for project name
- `scripts` for available commands

### Step 8: Detect Conflicts

Compare three sources for conflicts:
1. **CLAUDE.md content** (user-written)
2. **AF Memory memories** (stored in memory/memories.json)
3. **Actual project state** (from package.json, config files)

**Conflict detection rules:**

| Check | Conflict Example |
|-------|------------------|
| ORM mismatch | CLAUDE.md says "Prisma" but package.json has `drizzle-orm` |
| Framework mismatch | Memory says "React 18" but package.json has `react: ^19.0.0` |
| Auth mismatch | CLAUDE.md says "Clerk" but package.json has `next-auth` |
| Database mismatch | Memory says "MongoDB" but dependencies show `pg` |
| Package manager mismatch | CLAUDE.md says "yarn" but `pnpm-lock.yaml` exists |
| Styling mismatch | Memory says "styled-components" but `tailwindcss` is installed |

**Conflict output format:**
```
⚠️ Conflicts Detected

┌─────────────────────────────────────────────────────────────┐
│ CONFLICT 1: Database/ORM                                    │
├─────────────────────────────────────────────────────────────┤
│ CLAUDE.md says:     Prisma ORM                              │
│ package.json shows: drizzle-orm ^0.30.0                     │
│                                                             │
│ Action needed: Update CLAUDE.md or AF Memory              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ CONFLICT 2: React Version                                   │
├─────────────────────────────────────────────────────────────┤
│ AF Memory says:   React 18                                │
│ package.json shows: react ^19.0.0                           │
│                                                             │
│ Action needed: Run /afmem:save framework to update           │
└─────────────────────────────────────────────────────────────┘
```

**IMPORTANT:** Do NOT auto-resolve conflicts. List them and let the user decide.

### Step 9: Handle --check Flag

If `--check` flag is provided:
- Show conflicts detected
- Show what WOULD be imported (without importing)
- Show what WOULD be exported (without exporting)
- Exit without making any changes

```
🔍 Sync Check (dry run)

Would import from CLAUDE.md:
  • architecture: "Monorepo with Turborepo"
  • conventions: "Use server components by default"

Would skip (already in AF Memory):
  • stack
  • database

Conflicts detected: 2
  • Database: CLAUDE.md says Prisma, package.json shows Drizzle
  • React: Memory says 18, package.json shows 19

No changes made. Run /afmem:sync to apply.
```

### Step 10: Handle --import-only Flag

If `--import-only` flag is provided:
- Import new memories from CLAUDE.md
- Skip conflict detection (or show as warnings)
- Do NOT update CLAUDE.md
- Save memories to `memory/memories.json`

### Step 11: Handle --export-only Flag

If `--export-only` flag is provided:
- Skip import step
- Skip conflict detection
- Export current AF Memory memories to CLAUDE.md
- Same behavior as `/afmem:export`

### Step 12: Update CLAUDE.md (Full Sync)

After handling imports and showing conflicts, update the AF Memory section in CLAUDE.md:

1. **Preserve user content**: Keep everything outside the AF Memory section exactly as-is
2. **Find AF Memory section**: Look for `## Project Context (via AF Memory)`
3. **Replace AF Memory section**: Generate new content from current memories
4. **Write back**: Save the updated CLAUDE.md

**Generated AF Memory section format:**
```markdown
## Project Context (via AF Memory)

> Auto-generated by AF Memory. Last updated: [YYYY-MM-DD HH:MM]

### Tech Stack

| Key | Value |
|-----|-------|
| **stack** | Next.js 15, TypeScript, Drizzle, PostgreSQL |
| **framework** | Next.js 15 with App Router |
| **database** | Drizzle ORM with PostgreSQL |

### Preferences

| Key | Value |
|-----|-------|
| **styling** | Tailwind CSS v4 |
| **package-manager** | pnpm |

### Imported from CLAUDE.md

| Key | Value |
|-----|-------|
| **architecture** | Monorepo with Turborepo |
| **conventions** | Use server components by default |

---
```

Group memories by:
- **Tech Stack**: `stack`, `framework`, `database`, `auth`, `styling`, `testing`
- **Preferences**: `package-manager`, `conventions`, `preferences`
- **Imported from CLAUDE.md**: memories with tag `from-claude-md`
- **Other**: everything else

### Step 13: Write Updated Memories

Save the updated memories to `memory/memories.json`:

```bash
mkdir -p .afmem
```

Use the Write tool to save the JSON file.

### Step 14: Show Summary

Display a complete summary:

```
✅ AF Memory Sync Complete

┌─────────────────────────────────────────────────────────────┐
│ SUMMARY                                                     │
├─────────────────────────────────────────────────────────────┤
│ Memories imported from CLAUDE.md:  3                        │
│   • architecture                                            │
│   • conventions                                             │
│   • api-style                                               │
│                                                             │
│ Skipped (already exists):          2                        │
│   • stack                                                   │
│   • database                                                │
│                                                             │
│ Conflicts detected:                1                        │
│   ⚠️ React version mismatch (see above)                     │
│                                                             │
│ CLAUDE.md updated:                 Yes                      │
│ Total memories:                    8                        │
└─────────────────────────────────────────────────────────────┘

💡 Tips:
  • Fix conflicts with: /afmem:save <key> <correct-value>
  • View all memories: /afmem:list
  • Re-run after fixes: /afmem:sync --check
```

## Examples

### Example 1: Full Sync

User: `/afmem:sync`

Given CLAUDE.md:
```markdown
# My Project

## Architecture
We use a monorepo structure with Turborepo.
API routes follow REST conventions.

## Project Context (via AF Memory)
> Auto-generated by AF Memory. Last updated: 2024-01-15

| Key | Value |
|-----|-------|
| **stack** | Next.js 14, TypeScript, Prisma |

---

## Notes
Remember to run migrations before starting.
```

And package.json shows `drizzle-orm` instead of `prisma`.

Response:
```
🔄 Syncing AF Memory with CLAUDE.md...

📥 Importing from CLAUDE.md:
  ✓ architecture: "Monorepo structure with Turborepo"
  ✓ api-style: "REST conventions"
  ○ Skipped AF Memory section (auto-generated)

⚠️ Conflicts Detected

┌─────────────────────────────────────────────────────────────┐
│ CONFLICT: Database/ORM                                      │
├─────────────────────────────────────────────────────────────┤
│ AF Memory says:   Prisma                                  │
│ package.json shows: drizzle-orm                             │
│                                                             │
│ Fix with: /afmem:save database Drizzle ORM                   │
└─────────────────────────────────────────────────────────────┘

📤 Updated CLAUDE.md with latest memories

✅ Sync Complete
   Imported: 2 | Conflicts: 1 | Total memories: 5
```

### Example 2: Check Only

User: `/afmem:sync --check`

```
🔍 Sync Check (dry run)

Would import from CLAUDE.md:
  • architecture: "Monorepo with Turborepo"

Conflicts detected: 1
  ⚠️ ORM: Memory says "Prisma", package.json shows "drizzle-orm"

No changes made.
```

### Example 3: Import Only

User: `/afmem:sync --import-only`

```
📥 Importing from CLAUDE.md...

Imported:
  ✓ architecture: "Monorepo with Turborepo"
  ✓ api-conventions: "REST with versioning"

Skipped (already exists):
  ○ stack

✅ Import complete. 2 new memories saved.

Note: CLAUDE.md was not modified.
Run /afmem:sync to update CLAUDE.md with all memories.
```

## Error Handling

- **No CLAUDE.md**: Prompt to create one with `/afmem:export`
- **No memories.json**: Initialize empty memories
- **Parse errors**: Report the specific issue and line number if possible
- **Write permission**: Report error and suggest checking permissions
- **Empty user content**: Inform user there's nothing to import outside AF Memory section

## Conflict Resolution Tips

After detecting conflicts, suggest specific commands:

```
💡 To resolve conflicts:

  Database mismatch:
    /afmem:save database Drizzle ORM with PostgreSQL

  Framework version:
    /afmem:save framework Next.js 15 with App Router

  Then re-run:
    /afmem:sync --check
```

## Related Commands

- `/afmem:export` - Export memories to CLAUDE.md (same as --export-only)
- `/afmem:init` - Auto-detect project stack
- `/afmem:save` - Manually save a memory
- `/afmem:list` - View all memories
- `/afmem:recall` - Recall a specific memory
