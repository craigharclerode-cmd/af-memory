---
description: "List all saved memory keys"
argument-hint: "[--global|--project|--all]"
---

# /afmem:list

List all saved memory keys.

## Usage
/afmem:list
/afmem:list --global     # Show only global memories
/afmem:list --project    # Show only project memories
/afmem:list --all        # Show both (default)

## Instructions

When the user invokes `/afmem:list`, follow these steps:

### 1. Load Memories from Both Locations
```bash
# Check and read project memories
[ -f "memory/memories.json" ] && cat memory/memories.json

# Check and read global memories
[ -f "$HOME/memory/memories.json" ] && cat $HOME/memory/memories.json
```

### 2. Format Output

Group memories by source and display with agent tracking:
📚 AF Memory Index
── Project Memories (memory/memories.json) ──────────────────
KEY                     AGENT          UPDATED          PREVIEW
auth-flow               afmem-init      2026-02-11       NextAuth.js with JWT...
db-config               backend        2026-02-11       PostgreSQL via Prisma...
api-design              main           2026-02-10       RESTful, versioned...
Total: 3 memories
── Global Memories (~/memory/memories.json) ──────────────────
KEY                     AGENT          UPDATED          PREVIEW
preferred-editor        main           2026-02-08       VS Code with Vim...
Total: 1 memory
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total: 4 memories across all sources

### 3. Update MEMORY.md Index

**After listing, regenerate `memory/MEMORY.md` to keep the index current.**
```markdown
# AF Memory Index

> Auto-generated. Last updated: [YYYY-MM-DD HH:MM]

## Stored Memories

| Key | Preview | Agent | Updated |
|-----|---------|-------|---------|
| auth-flow | NextAuth.js with JWT... | afmem-init | 2026-02-11 |
| db-config | PostgreSQL via Prisma... | backend | 2026-02-11 |

## Files

| File | Purpose |
|------|---------|
| `memories.json` | All stored memories |
| `access.log` | Read/write audit trail |
| `config.json` | Plugin configuration |
| `MEMORY.md` | This index file |
```

### 4. Empty State

If no memories exist:
📚 AF Memory Index
No memories saved yet.
Get started:
/afmem:save <key> <value>  - Save your first memory
Examples:
/afmem:save user-prefers-typescript User prefers TypeScript over JavaScript
/afmem:save project-uses-nextjs This project uses Next.js 14 with App Router

### 5. Sorting

Default sort: by `updated` date (most recent first)

## Output Fields
- **KEY**: The memory identifier
- **AGENT**: Who wrote this memory (main, afmem-init, backend, frontend, etc.)
- **UPDATED**: Last modified date (relative or absolute)
- **PREVIEW**: First 40 characters of the value

## Tips to Show User
After listing, remind user:
- Use `/afmem:recall <key>` to see full memory content
- Use `/afmem:forget <key>` to delete a memory
- Use `/afmem:context <keyword>` to search by topic
- Use `/afmem:log` to see access audit trail
