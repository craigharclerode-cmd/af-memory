---
description: "Enable or disable automatic activity capture"
argument-hint: "[on|off|status]"
---

# /afmem-pro:auto-capture

Toggle automatic activity capture on or off.

## Usage
```
/afmem-pro:auto-capture on      # Enable auto-capture
/afmem-pro:auto-capture off     # Disable auto-capture
/afmem-pro:auto-capture status  # Check current status
/afmem-pro:auto-capture         # Show status (same as status)
```

## Arguments
- `on`: Enable automatic activity capture for this project
- `off`: Disable automatic activity capture
- `status`: Show current auto-capture status (default if no argument)

## Instructions

When the user invokes `/afmem-pro:auto-capture`, follow these steps:

### 1. Parse Argument
Extract the action: `on`, `off`, or `status` (default to `status` if empty).

### 2. Configuration File Location
Auto-capture config is stored in: `memory/pro/config.json`

```json
{
  "autoCapture": {
    "enabled": true,
    "tools": ["Edit", "Write", "Bash"],
    "capturePatterns": {
      "Edit": "file modifications",
      "Write": "new files created",
      "Bash": "git commits, npm/bun commands"
    }
  }
}
```

### 3. Handle Actions

**For `on`:**
```bash
mkdir -p memory/pro
```

Read or create `memory/pro/config.json`, set `autoCapture.enabled = true`, write back.

Confirm:
```
Auto-capture ENABLED

What will be captured:
  - Edit: File modifications
  - Write: New files created
  - Bash: Git commits, npm/bun commands

Activities saved to: memory/pro/activity.log
Review with: /afmem-pro:activity
```

**For `off`:**
Read `memory/pro/config.json`, set `autoCapture.enabled = false`, write back.

Confirm:
```
Auto-capture DISABLED

No automatic activity capture will occur.
```

**For `status`:**
Read `memory/pro/config.json` and display current state:

```
Auto-capture Status

  Enabled: Yes/No
  Tools monitored: Edit, Write, Bash
  Activity log: memory/pro/activity.log
  Entries captured: N

Commands:
  /afmem-pro:auto-capture on   - Enable
  /afmem-pro:auto-capture off  - Disable
  /afmem-pro:activity          - View captured activities
```

### 4. Initialize Config (if not exists)

If `memory/pro/config.json` doesn't exist, create it:

```json
{
  "version": "1.0",
  "autoCapture": {
    "enabled": false,
    "tools": ["Edit", "Write", "Bash"],
    "capturePatterns": {
      "Edit": "file modifications",
      "Write": "new files created",
      "Bash": "git commits, npm/bun commands"
    },
    "excludePaths": [
      "node_modules/**",
      ".git/**",
      "*.log",
      "memory/pro/**"
    ]
  }
}
```

### 5. Read/Write Config

Use the Read tool to check for existing config, then Write tool to update it.

## Example Interactions

### Enable auto-capture
User: `/afmem-pro:auto-capture on`

```
Auto-capture ENABLED

What will be captured:
  - Edit: File modifications
  - Write: New files created
  - Bash: Git commits, npm/bun commands

Activities saved to: memory/pro/activity.log
Review captured activities: /afmem-pro:activity
```

### Disable auto-capture
User: `/afmem-pro:auto-capture off`

```
Auto-capture DISABLED

No automatic activity capture will occur.
```

### Check status
User: `/afmem-pro:auto-capture status`

```
Auto-capture Status

  Enabled: Yes
  Tools: Edit, Write, Bash
  Log: memory/pro/activity.log (12 entries)

Commands:
  /afmem-pro:auto-capture off  - Disable
  /afmem-pro:activity          - View log
```

## Related Commands
- `/afmem-pro:activity` - View captured activity log
- `/afmem-pro:clear` - Clear activity log
