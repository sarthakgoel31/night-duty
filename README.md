# Night Duty

**Keep your Mac awake with the screen blacked out.**

## What It Does

Night Duty is a Mac utility skill for overnight tasks -- long-running builds, downloads, data syncs, or anything that needs your laptop awake but not burning your retinas. It has two modes: caffeinate (prevent sleep indefinitely) and blind (dim screen to absolute zero brightness). Use them separately or together. An hourly watcher sends macOS notifications so you do not forget it is running.

## How It Works

**Caffeinate mode:**
1. Runs `caffeinate -dis` in the background (prevents display sleep, idle sleep, and system sleep)
2. Saves the PID to `/tmp/caffeinate.pid` for clean shutdown later
3. Starts an hourly watcher script that sends macOS notifications as reminders
4. Say "stop caffeinate" to kill it cleanly

**Blind mode:**
1. Uses `osascript` to send 50 brightness-down key events (key code 145), guaranteeing zero brightness from any starting level
2. Keyboard backlight must be dimmed manually via F5 (macOS does not allow programmatic keyboard backlight control)
3. Say "restore" or "unblind" to bring brightness back to approximately 50%

**Night duty mode (both):**
1. Starts caffeinate with a blind-aware watcher that delays hourly notifications until the screen is restored
2. Dims screen to zero
3. When you say "restore", screen comes back and the hourly reminders begin
4. Say "stop night duty" to shut everything down

## Key Features

- **Two independent modes:** Use caffeinate alone, blind alone, or both together
- **Hourly reminders:** macOS notifications every hour so you remember to stop it
- **Blind-aware watcher:** When running both modes, hourly reminders wait until screen brightness is restored before counting
- **Clean shutdown:** All PIDs are tracked in `/tmp/` for reliable cleanup
- **No confirmation prompts:** Executes immediately on command -- no "are you sure?" dialogs

## Usage

```
caffeinate          # Keep Mac awake, screen stays on
blind               # Dim screen to zero, say "restore" to bring it back
night duty          # Interactive -- asks which mode(s) you want
restore             # Bring screen brightness back to ~50%
stop caffeinate     # Kill the caffeinate process
stop night duty     # Kill everything (caffeinate + watcher + blind state)
```

**Trigger phrases:** "night duty", "caffeinate", "blind"

## Technical Details

- **Key code 145:** macOS brightness down (50 repeats hits absolute zero from max)
- **Key code 144:** macOS brightness up (8 repeats restores to approximately 50%)
- **caffeinate flags:** `-d` (prevent display sleep), `-i` (prevent idle sleep), `-s` (prevent system sleep)
- **State files:** `/tmp/caffeinate.pid`, `/tmp/night-duty-watcher.pid`, `/tmp/night-duty-blind`
- **Limitation:** Keyboard backlight cannot be controlled programmatically on macOS -- manual F5 key required

---

Built with [Claude Code](https://claude.ai/code) as a slash command skill.
