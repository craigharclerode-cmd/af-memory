---
description: "Enable or disable automatic syncing to CLAUDE.md"
argument-hint: "[on|off|status]"
---

# /afmem:auto-sync

Toggle automatic syncing of memories to CLAUDE.md.

## Usage
```
/afmem:auto-sync on      # Enable auto-sync
/afmem:auto-sync off     # Disable auto-sync
/afmem:auto-sync status  # Check current status
/afmem:auto-sync         # Show status (same as status)
```

## Arguments
- `on`: Enable automatic syncing to CLAUDE.md after memory changes
- `off`: Disable automatic syncing
- `status`: Show current auto-sync status (default if no argument)

## Instructions

When the user invokes `/afmem:auto-sync`, follow these steps:

### 1. Parse Argument
Extract the action: `on`, `off`, or `status` (default to `status` if empty).

### 2. Configuration File Location
Auto-sync config is stored in: `memory/config.json`

```json
{
  "autoSync": true
}
```

### 3. Handle Actions

**For `on`:**
```bash
mkdir -p .afmem
```

Read or create `memory/config.json`, set `autoSync = true`, write back.

Confirm:
```
Auto-sync ENABLED

What happens now:
  - /afmem:save   -> CLAUDE.md updates automatically
  - /afmem:init   -> CLAUDE.md updates automatically
  - /afmem:forget -> CLAUDE.md updates automatically

CLAUDE.md will always stay in sync with your memories.
```

**For `off`:**
Read `memory/config.json`, set `autoSync = false`, write back.

Confirm:
```
Auto-sync DISABLED

CLAUDE.md will not update automatically.
Run these manually when needed:
  - /afmem:export  (write memories to CLAUDE.md)
  - /afmem:sync    (two-way sync)
```

**For `status`:**
Read `memory/config.json` and display current state:

```
Auto-sync Status

  Enabled: Yes/No
  Config: memory/config.json

When enabled, these commands update CLAUDE.md:
  - /afmem:save
  - /afmem:init
  - /afmem:forget

Commands:
  /afmem:auto-sync on   - Enable
  /afmem:auto-sync off  - Disable
```

### 4. Initialize Config (if not exists)

If `memory/config.json` doesn't exist, create it:

```json
{
  "autoSync": false
}
```

### 5. Read/Write Config

Use the Read tool to check for existing config, then Write tool to update it.

**Reading config:**
```bash
[ -f "memory/config.json" ] && cat memory/config.json
```

**Merging config:**
When updating, preserve any existing keys (like `autoCapture`) and only update `autoSync`.

## Example Interactions

### Enable auto-sync
User: `/afmem:auto-sync on`

```
Auto-sync ENABLED

What happens now:
  - /afmem:save   -> CLAUDE.md updates automatically
  - /afmem:init   -> CLAUDE.md updates automatically
  - /afmem:forget -> CLAUDE.md updates automatically

CLAUDE.md will always stay in sync with your memories.
```

### Disable auto-sync
User: `/afmem:auto-sync off`

```
Auto-sync DISABLED

CLAUDE.md will not update automatically.
Run these manually when needed:
  - /afmem:export  (write memories to CLAUDE.md)
  - /afmem:sync    (two-way sync)
```

### Check status
User: `/afmem:auto-sync status`

```
Auto-sync Status

  Enabled: Yes
  Config: memory/config.json

When enabled, these commands update CLAUDE.md:
  - /afmem:save
  - /afmem:init
  - /afmem:forget

Commands:
  /afmem:auto-sync off  - Disable
```

## Related Commands
- `/afmem:export` - Manually export memories to CLAUDE.md
- `/afmem:sync` - Two-way sync between memories and CLAUDE.md
- `/afmem:save` - Save a memory (triggers auto-sync if enabled)
- `/afmem:init` - Initialize project (triggers auto-sync if enabled)
- `/afmem:forget` - Delete a memory (triggers auto-sync if enabled)
