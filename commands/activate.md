---
description: "Activate AF Memory Pro with your license key"
argument-hint: "<license-key>"
---

# /afmem:activate

Activate AF Memory Pro by entering your license key. Once activated, all Pro features are unlocked — Cortex, Foresight, Decay, Import, cross-provider Export, and advanced Health diagnostics.

## Usage
```
/afmem:activate AFMEM-PRO-XXXX-XXXX
```

## Instructions

### Step 1: Parse the License Key

Extract the license key from the argument. If no argument is provided, show:

```
Usage: /afmem:activate <license-key>

License keys look like: AFMEM-PRO-XXXX-XXXX
Get yours at afmem.dev/pro
```

Stop.

### Step 2: Validate Key Format

Check the key format:
- Must start with `AFMEM-PRO-`
- Followed by exactly 9 characters (letters, numbers, or hyphens)

If the format is invalid:

```
❌ Invalid license key format.

Keys look like: AFMEM-PRO-XXXX-XXXX
Get yours at afmem.dev/pro
```

Stop.

### Step 3: Save License

Create or overwrite `memory/license.json` with:

```json
{
  "key": "<license-key>",
  "plan": "pro",
  "activated_at": "<current ISO-8601 timestamp>"
}
```

### Step 4: Confirm Activation

```
✅ AF Memory Pro activated!

All Pro features are now unlocked:
  /afmem:cortex      Memory intelligence layer
  /afmem:foresight   Predictive context loading
  /afmem:decay       Auto-archive stale memories
  /afmem:import      Cross-provider import
  /afmem:export --codex / --cursor / --windsurf / --all
  /afmem:health --verbose / --fix

License saved to memory/license.json
```

## Related Commands

- `/afmem:health` — Check system status
- `/afmem:cortex` — Memory intelligence (now unlocked)
- `/afmem:foresight` — Predictive loading (now unlocked)
