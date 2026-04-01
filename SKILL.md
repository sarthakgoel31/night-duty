---
name: night-duty
description: "Use when someone says 'night duty', 'caffeinate', or 'blind'. Keeps Mac awake or dims screen to zero."
---

## What This Skill Does

Mac utility skill with three modes:
- **caffeinate** — keeps the Mac awake indefinitely (prevents sleep)
- **blind** — dims screen to zero, say "restore" to bring it back to ~50%. Keyboard backlight must be dimmed manually (F5 key)
- **night duty** — asks which of the two (or both) and executes

## Detecting the Mode

Determine mode from the user's message:
- If the message contains "caffeinate" (and NOT "night duty") → run **Caffeinate Mode** directly
- If the message contains "blind" (and NOT "night duty") → run **Blind Mode** directly
- If the message contains "night duty" → run **Night Duty Mode** (interactive)

## Caffeinate Mode

Run this bash command in the background:

```bash
caffeinate -dis &
CAFFEINE_PID=$!
echo $CAFFEINE_PID > /tmp/caffeinate.pid
# Start hourly watcher (no --wait-blind since this is caffeinate-only)
nohup /Users/sarthak/Claude/.claude/scripts/night-duty/watcher.sh > /dev/null 2>&1 &
echo $! > /tmp/night-duty-watcher.pid
echo "Caffeinate running (PID: $CAFFEINE_PID). Hourly reminders active."
```

Use `run_in_background: true` on the Bash tool so it doesn't block.

Tell the user:
- Mac will stay awake until they say "stop caffeinate" or "kill caffeinate"
- They'll get a macOS notification every hour as a reminder

## Blind Mode

Two-step process: dim now, restore when user says "restore".

**Step 1 — Dim screen:**

```bash
osascript -e '
tell application "System Events"
    repeat 50 times
        key code 145
        delay 0.03
    end repeat
end tell' && touch /tmp/night-duty-blind && echo "Screen dimmed to zero."
```

Tell the user: "Screen dimmed. Dim your keyboard manually (F5 key). Say 'restore' when you want brightness back."

**Step 2 — Restore (when user says "restore", "unblind", or "lights on"):**

```bash
osascript -e '
tell application "System Events"
    repeat 8 times
        key code 144
        delay 0.05
    end repeat
end tell' && rm -f /tmp/night-duty-blind && echo "Brightness restored to ~50%."
```

## Night Duty Mode (Interactive)

Ask the user: "Night duty activated. What do you need?"
- **Caffeinate** — keep Mac awake
- **Blind** — dim screen to zero
- **Both** — caffeinate + blind

Then execute the chosen mode(s). If both:

1. Run caffeinate with the blind-aware watcher:

```bash
caffeinate -dis &
CAFFEINE_PID=$!
echo $CAFFEINE_PID > /tmp/caffeinate.pid
touch /tmp/night-duty-blind
# Start watcher in --wait-blind mode (waits for keypress restore before counting hours)
nohup /Users/sarthak/Claude/.claude/scripts/night-duty/watcher.sh --wait-blind > /dev/null 2>&1 &
echo $! > /tmp/night-duty-watcher.pid
echo "Caffeinate running (PID: $CAFFEINE_PID). Hourly reminders start after screen restores."
```

2. Then run blind mode (the `rm -f /tmp/night-duty-blind` in the restore step signals the watcher to start counting).

## Stopping Caffeinate

If the user says "stop caffeinate", "kill caffeinate", or "stop night duty":

```bash
kill $(cat /tmp/caffeinate.pid) 2>/dev/null && rm -f /tmp/caffeinate.pid
kill $(cat /tmp/night-duty-watcher.pid) 2>/dev/null && rm -f /tmp/night-duty-watcher.pid
rm -f /tmp/night-duty-blind
echo "Night duty fully stopped."
```

## Notes

- Key code 145 = screen brightness down, key code 144 = screen brightness up (macOS system keys)
- 50 repeats for dimming ensures we hit absolute zero from any starting brightness
- 8 repeats for screen restore brings it to ~50% — user can adjust from there
- Keyboard backlight cannot be controlled programmatically on this Mac — user must dim manually via F5 key
- caffeinate flags: -d (prevent display sleep), -i (prevent idle sleep), -s (prevent system sleep)
- Blind mode is two-step: dim immediately, restore when user says "restore"/"unblind"/"lights on"
- Do NOT ask for confirmation before running bash commands — just execute
