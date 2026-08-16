---
description: "Export AF Memory memories to Codex CLI, Cursor, and Windsurf rule files"
argument-hint: "[--codex | --cursor | --windsurf | --all | --status]"
---

# /afmem-pro:export

Export project memories to AI tool rule files used by Codex CLI (AGENTS.md), Cursor (.cursor/rules/afmem-memory.mdc), and Windsurf (.windsurfrules). Keeps your entire AI toolchain in sync from a single source of truth: `memory/memories.json`.

## Usage
```
/afmem-pro:export --codex      # Generate AGENTS.md for Codex CLI
/afmem-pro:export --cursor     # Generate .cursor/rules/afmem-memory.mdc for Cursor
/afmem-pro:export --windsurf   # Generate .windsurfrules for Windsurf
/afmem-pro:export --all        # Generate all three + sync CLAUDE.md
/afmem-pro:export --status     # Check which export files exist
```

## Arguments
- `--codex` (optional): Export to `AGENTS.md` in project root (Codex CLI format)
- `--cursor` (optional): Export to `.cursor/rules/afmem-memory.mdc` (MDC frontmatter format)
- `--windsurf` (optional): Export to `.windsurfrules` in project root (plain markdown)
- `--all` (optional): Run all three exports and trigger CLAUDE.md sync
- `--status` (optional): Report which export files exist and their last modified timestamps

## Instructions

When the user invokes `/afmem-pro:export`, follow these steps:

### Step 1: Parse Arguments

Extract the subcommand from the user's input:

```
--codex      → Export to AGENTS.md
--cursor     → Export to .cursor/rules/afmem-memory.mdc
--windsurf   → Export to .windsurfrules
--all        → Run all three exports + CLAUDE.md sync
--status     → Show export file status
(no args)    → Show Usage block above and stop
```

If no argument is provided or the argument is unrecognised, display the Usage block above and stop.

---

## Memory Reading Logic (used by all export subcommands)

Before generating any export file, perform these steps to load and prepare memories. This logic is shared by `--codex`, `--cursor`, `--windsurf`, and `--all`.

### Step 2: Read memories.json

Read `memory/memories.json` using the Read tool:

```bash
[ -f "memory/memories.json" ] && cat memory/memories.json
```

If the file does not exist, stop and show:

```
❌ No AF Memory memories found. Run /afmem:init first.

   To initialize: /afmem:init
   To save a memory: /afmem:save <key> <value>
```

### Step 3: Parse the Memories Array

The file uses the new array format:

```json
{
  "memories": [
    {
      "key": "version",
      "value": "0.3.0",
      "created": "2026-02-26T16:00:00.000Z",
      "updated": "2026-02-26T16:00:00.000Z",
      "projectPath": "C:/Users/...",
      "agent_id": "afmem-init",
      "tags": ["auto-detected"]
    }
  ]
}
```

**Backward compatibility:** Some older files store memories as a flat object (keys at the top level):

```json
{
  "stack": { "value": "Next.js", "created": "...", "updated": "..." },
  "auth":  { "value": "NextAuth", "created": "...", "updated": "..." }
}
```

If the parsed JSON does not have a `memories` array, treat each top-level key as a memory entry, converting it to the array format internally:

```
key = the JSON key name
value = entry.value
created = entry.created
updated = entry.updated
type = entry.type (if present, else undefined)
vitality = entry.vitality (if present, else undefined)
confidence = entry.confidence (if present, else undefined)
```

### Step 4: Filter Out Extinct Memories

Skip any memory where `vitality.state === "extinct"`. These memories are considered dead and must not appear in export files.

For Cortex-enhanced memories, `vitality` is an object. For plain memories, `vitality` is absent — treat absent vitality as active (do not skip).

### Step 5: Group by Type

Group the remaining memories by their `type` field. Valid type values are: `"fact"`, `"rule"`, `"procedure"`, `"preference"`, `"warning"`, `"decision"`. Memories without a `type` field go under `"Other"`.

**Group order for output (always in this sequence, skip empty groups):**

1. Facts
2. Rules
3. Procedures
4. Preferences
5. Warnings
6. Decisions
7. Other

Do not emit a section header for a group that has zero memories.

### Step 6: Sort Within Groups

Within each group, sort memories alphabetically by `key` (case-insensitive, A → Z).

### Step 7: Token Budget Check

After grouping, estimate the output size. Target under 4000 tokens total (approximately 16 000 characters). If the estimated output would exceed this, warn the user at the end of the confirmation message but proceed with full export. Do not truncate content silently.

---

## /afmem-pro:export --codex

Generate `AGENTS.md` in the project root for Codex CLI.

### Step 8a: Build AGENTS.md Content

Using the prepared groups from Steps 2-7, generate the following markdown:

```
# Project Memory (powered by AF Memory)
> Auto-generated. Do not edit manually. Source: memory/memories.json
> Last exported: <ISO timestamp> | Memories: <count> | AF Memory Pro

## Facts
- <key>: <value>
- <key>: <value>

## Rules
- <key>: <value>

## Procedures
- <key>: <value>

## Preferences
- <key>: <value>

## Warnings
- <key>: <value>

## Decisions
- <key>: <value>

## Other
- <key>: <value>

---
To update: save memories with /afmem:save in Claude Code, then /afmem-pro:export --codex
To write back: add entries to memory/memories.json following existing format
```

- Replace `<ISO timestamp>` with the current UTC time in ISO-8601 format.
- Replace `<count>` with the number of memories being exported (after filtering extinct).
- Only include section headers for groups that have at least one memory.
- Each bullet line is `- <key>: <value>`.

### Step 9a: Write AGENTS.md

Write the generated content to `AGENTS.md` in the project root using the Write tool.

### Step 10a: Confirm to User

Display:

```
✅ Exported 12 memories to AGENTS.md (Codex CLI)
   Path: ./AGENTS.md
   Size: ~847 bytes (~212 tokens)

💡 Codex CLI reads AGENTS.md automatically from repo root.
   Run /afmem-pro:export --codex after saving new memories.
```

Calculate approximate bytes as the character count of the generated file content. Calculate approximate tokens as `round(bytes / 4)`.

---

## /afmem-pro:export --cursor

Generate `.cursor/rules/afmem-memory.mdc` in the project root for Cursor.

### Step 8b: Ensure Directory Exists

Check whether `.cursor/rules/` exists and create it if not:

```bash
mkdir -p .cursor/rules
```

### Step 9b: Build MDC Content

Using the prepared groups from Steps 2-7, generate the following MDC file (YAML frontmatter + markdown body):

```
---
description: Project memory and coding context managed by AF Memory
globs: ["**/*"]
alwaysApply: true
---

# Project Memory (powered by AF Memory)

## Facts
- <key>: <value>

## Rules
- <key>: <value>

## Procedures
- <key>: <value>

## Preferences
- <key>: <value>

## Warnings
- <key>: <value>

## Decisions
- <key>: <value>

## Other
- <key>: <value>

<!-- Generated by AF Memory Pro. Source: memory/memories.json -->
<!-- Last exported: <ISO timestamp> | Memories: <count> -->
```

- Only include section headers for groups with at least one memory.
- The `<!-- -->` comment lines are always placed at the very end of the file.

### Step 10b: Write the MDC File

Write the generated content to `.cursor/rules/afmem-memory.mdc` using the Write tool.

### Step 11b: Confirm to User

Display:

```
✅ Exported 12 memories to .cursor/rules/afmem-memory.mdc (Cursor)
   Path: ./.cursor/rules/afmem-memory.mdc
   Size: ~863 bytes (~216 tokens)

💡 Cursor loads .mdc files with alwaysApply: true at session start.
   Run /afmem-pro:export --cursor after saving new memories.
```

---

## /afmem-pro:export --windsurf

Generate `.windsurfrules` in the project root for Windsurf.

### Step 8c: Build .windsurfrules Content

Using the prepared groups from Steps 2-7, generate plain markdown with no YAML frontmatter:

```
# Project Memory (powered by AF Memory)
> Auto-generated. Do not edit manually. Source: memory/memories.json
> Last exported: <ISO timestamp> | Memories: <count> | AF Memory Pro

## Facts
- <key>: <value>

## Rules
- <key>: <value>

## Procedures
- <key>: <value>

## Preferences
- <key>: <value>

## Warnings
- <key>: <value>

## Decisions
- <key>: <value>

## Other
- <key>: <value>

---
To update: save memories with /afmem:save in Claude Code, then /afmem-pro:export --windsurf
```

- Only include section headers for groups with at least one memory.

### Step 9c: Write .windsurfrules

Write the generated content to `.windsurfrules` in the project root using the Write tool.

### Step 10c: Confirm to User

Display:

```
✅ Exported 12 memories to .windsurfrules (Windsurf)
   Path: ./.windsurfrules
   Size: ~821 bytes (~205 tokens)

💡 Windsurf reads .windsurfrules from the repo root automatically.
   Run /afmem-pro:export --windsurf after saving new memories.
```

---

## /afmem-pro:export --all

Run all three exports and also sync CLAUDE.md.

### Step 8d: Run All Exports in Sequence

Execute the following in order:

1. **Codex export**: perform Steps 8a–10a to generate `AGENTS.md`.
2. **Cursor export**: perform Steps 8b–11b to generate `.cursor/rules/afmem-memory.mdc`.
3. **Windsurf export**: perform Steps 8c–10c to generate `.windsurfrules`.
4. **CLAUDE.md sync**: generate the AF Memory section of `CLAUDE.md` from memories (same logic as `/afmem:export`). Update only the AF Memory section; preserve all other content.

### Step 9d: Confirm to User

Display a combined summary:

```
✅ AF Memory Pro — All exports complete

   AGENTS.md                         ./AGENTS.md             ~847 bytes
   .cursor/rules/afmem-memory.mdc     ./.cursor/rules/...     ~863 bytes
   .windsurfrules                    ./.windsurfrules        ~821 bytes
   CLAUDE.md                         ./CLAUDE.md             updated

   Memories exported: 12
   Extinct skipped:   2

💡 All AI tools are now in sync with memory/memories.json.
   Run /afmem-pro:export --all after any /afmem:save to refresh all targets.
```

---

## /afmem-pro:export --status

Check which export files exist and their last modified timestamps. Do not modify any files.

### Step 8e: Check File Existence and Timestamps

Run a combined existence and timestamp check:

```bash
ls -l AGENTS.md .cursor/rules/afmem-memory.mdc .windsurfrules 2>/dev/null
```

For each file, note whether it exists and extract its last-modified timestamp.

### Step 9e: Read Auto-Export Config (Optional)

Check whether `memory/pro/config.json` exists:

```bash
[ -f "memory/pro/config.json" ] && cat memory/pro/config.json || echo "NO_PRO_CONFIG"
```

If the file exists, read the `autoExport` settings to determine which targets are enabled.

### Step 10e: Display Status

```
Export Status

  AGENTS.md                        ✅ exists   Last updated: 2026-03-01 14:23
  .cursor/rules/afmem-memory.mdc    ❌ not found
  .windsurfrules                   ✅ exists   Last updated: 2026-03-01 14:23

Auto-export: Enabled → targets: codex, windsurf
```

If `memory/pro/config.json` does not exist or has no `autoExport` key:

```
Auto-export: Not configured
```

If all three files are missing:

```
No export files found. Run /afmem-pro:export --all to generate all targets.
```

---

## Example Interactions

### Example 1: Export to Codex CLI

User: `/afmem-pro:export --codex`

Given `memory/memories.json` with 7 active memories (2 facts, 2 preferences, 3 other):

```
✅ Exported 7 memories to AGENTS.md (Codex CLI)
   Path: ./AGENTS.md
   Size: ~650 bytes (~163 tokens)

💡 Codex CLI reads AGENTS.md automatically from repo root.
   Run /afmem-pro:export --codex after saving new memories.
```

Generated `AGENTS.md`:

```markdown
# Project Memory (powered by AF Memory)
> Auto-generated. Do not edit manually. Source: memory/memories.json
> Last exported: 2026-03-02T09:15:00.000Z | Memories: 7 | AF Memory Pro

## Facts
- project-name: AF Memory — Claude Code Plugin for persistent local memory
- version: 0.3.0

## Preferences
- favorite-language: TypeScript
- preferred-style: functional

## Other
- author: Craig Harclerode
- database: PostgreSQL
- package-manager: npm

---
To update: save memories with /afmem:save in Claude Code, then /afmem-pro:export --codex
To write back: add entries to memory/memories.json following existing format
```

---

### Example 2: Check Export Status

User: `/afmem-pro:export --status`

```
Export Status

  AGENTS.md                        ✅ exists   Last updated: 2026-03-02 09:15
  .cursor/rules/afmem-memory.mdc    ❌ not found
  .windsurfrules                   ✅ exists   Last updated: 2026-03-02 09:15

Auto-export: Enabled → targets: codex, windsurf
```

---

### Example 3: Export All Targets

User: `/afmem-pro:export --all`

Given 12 active memories (2 extinct skipped):

```
✅ AF Memory Pro — All exports complete

   AGENTS.md                         ./AGENTS.md             ~1024 bytes
   .cursor/rules/afmem-memory.mdc     ./.cursor/rules/...      ~1051 bytes
   .windsurfrules                    ./.windsurfrules          ~998 bytes
   CLAUDE.md                         ./CLAUDE.md             updated

   Memories exported: 12
   Extinct skipped:   2

💡 All AI tools are now in sync with memory/memories.json.
   Run /afmem-pro:export --all after any /afmem:save to refresh all targets.
```

---

### Example 4: No memories.json Found

User: `/afmem-pro:export --codex`

```
❌ No AF Memory memories found. Run /afmem:init first.

   To initialize: /afmem:init
   To save a memory: /afmem:save <key> <value>
```

---

## Error Handling

- **No memory/memories.json**: Show init prompt (see Step 2) and stop.
- **Empty memories after filtering extinct**: Export an empty file with only the header and footer lines. Inform user: `⚠️ All memories are extinct or no memories exist. Exported empty file.`
- **Write permission error**: Report the OS error and suggest checking file permissions.
- **Malformed memories.json (invalid JSON)**: Report: `❌ memories.json is not valid JSON. Check the file for syntax errors.` and stop.
- **Old flat-object format**: Handle transparently via the backward-compatibility logic in Step 3.
- **.cursor/rules/ directory cannot be created**: Report error and suggest creating the directory manually.

## Implementation Notes

**Extinct filtering is mandatory.** Never export a memory with `vitality.state === "extinct"` regardless of its other fields.

**Section headers are conditional.** Only emit a `## Facts` heading (or any other group heading) if that group contains at least one non-extinct memory. Do not emit empty sections.

**Alphabetical sort is case-insensitive.** Sort by `key.toLowerCase()` within each group.

**Token estimate formula:** `tokens ≈ round(character_count / 4)`. Use length of the final file content string, not the raw JSON.

**Write tool usage:** Use the Write tool for creating or overwriting export files. Never use Bash redirection or echo for file writes.

**Directory creation for Cursor:** Always run `mkdir -p .cursor/rules` before writing the MDC file, even if it appears to already exist, to avoid path errors.

**CLAUDE.md sync in --all:** Follow the same section-detection logic as `/afmem:export` — find and replace only the `## Project Context (via AF Memory)` section, preserving all other content.

## Related Commands

- `/afmem:export` — Export memories to CLAUDE.md (open-source, Claude Code only)
- `/afmem:save` — Save or update a memory
- `/afmem:init` — Auto-detect project stack and initialize memories
- `/afmem:list` — View all memory keys and values
- `/afmem:forget` — Remove a memory
- `/afmem:cortex` — Memory intelligence layer (manages vitality and extinct state)
- `/afmem-pro:auto-capture` — Toggle auto-capture of agent activity
- `/afmem-pro:activity` — View or manage the captured activity log
