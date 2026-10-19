# zoom-quiet

[![ci](https://github.com/chaenmasahiro0425/zoom-quiet/actions/workflows/ci.yml/badge.svg)](https://github.com/chaenmasahiro0425/zoom-quiet/actions/workflows/ci.yml)
![macOS](https://img.shields.io/badge/platform-macOS-black)
![License: MIT](https://img.shields.io/badge/License-MIT-blue)

English | [日本語](README.md)

**A tiny macOS tool that silences Claude notifications only while you are in a Zoom meeting.**

When you join a meeting, zoom-quiet turns on a macOS Focus; when you leave, it turns it off again.
Set that Focus to silence the Claude apps, and Claude Code notifications (desktop app and CLI) stay quiet during meetings only — no banners in your screen share, no sounds while you talk.

![Joining turns the Focus on, leaving turns it off](docs/media/demo-en.gif)

▶ **Watch the 30-second intro video**: https://chaenmasahiro0425.github.io/zoom-quiet/en/

- No dependencies (only macOS built-ins: bash, pgrep, netstat, shortcuts)
- Starts at login via launchd and checks every 5 seconds (practically 0% CPU)
- Only turns off the Focus it turned on itself; other Focus modes are never touched
- You choose which apps to silence in the Focus settings (Claude, Slack, your terminal, …)

## How it works

![How zoom-quiet works](docs/media/how-it-works-en.png)

```
launchd ─▶ zoom-quiet run (every 5 s)
              │
              ├─ In a meeting?  CptHost / aomhost is running, or zoom.us has UDP sockets
              │
              ├─ Joined  → shortcuts run "Zoom Quiet On"   → Focus "Zoom Quiet" on
              └─ Left    → shortcuts run "Zoom Quiet Off"  → Focus "Zoom Quiet" off
```

When you join a meeting, Zoom spawns the helper processes `CptHost` and `aomhost` and starts streaming audio/video over UDP.
None of these appear outside a meeting (measured on macOS 26.6). Any one of them counts as "in a meeting".

macOS has no command to toggle a Focus directly, so zoom-quiet runs two Shortcuts.

## Install

```bash
git clone https://github.com/chaenmasahiro0425/zoom-quiet.git ~/zoom-quiet
```

1. Create a Focus and two Shortcuts (one time, ~3 minutes) — step by step in [docs/setup.en.md](docs/setup.en.md):
   - **Focus** `Zoom Quiet`: System Settings → Focus → Add Focus → Custom. Choose **Silence notifications from** and add the Claude app(s) — and your terminal if you want CLI banners silenced too.
   - **Shortcut `Zoom Quiet On`**: one action, *Set Focus* → `Zoom Quiet` → **On** (until turned off).
   - **Shortcut `Zoom Quiet Off`**: one action, *Set Focus* → `Zoom Quiet` → **Off**.
   - Prefer other names? Set `ON_SHORTCUT` / `OFF_SHORTCUT` in the config (see below).
   - Messages and default names follow your macOS language (English or Japanese). Force one with `UI_LANG=en`.
2. Enable autostart:

```bash
~/zoom-quiet/bin/zoom-quiet install
```

You are ready when every line of the `doctor` output shows ✓.

## Usage

Nothing to do day to day. Join a Zoom meeting and the Focus turns on within seconds; leave and it turns off.

```bash
zoom-quiet status      # in a meeting? is the Focus on (by zoom-quiet)?
zoom-quiet check       # exit code 0 while in a meeting (for scripts and hooks)
zoom-quiet on / off    # toggle the Focus by hand (for testing)
zoom-quiet doctor      # check the setup
zoom-quiet uninstall   # stop autostart
```

Logs go to `~/.local/state/zoom-quiet/zoom-quiet.log`.

### With Claude Code hooks

If a Claude Code hook plays a sound, skip it during meetings (`~/.claude/settings.json`):

```json
{
  "hooks": {
    "Stop": [
      { "hooks": [ { "type": "command",
        "command": "~/zoom-quiet/bin/zoom-quiet check -q || afplay /System/Library/Sounds/Glass.aiff",
        "async": true } ] }
    ]
  }
}
```

## Configuration

`~/.config/zoom-quiet/config` (created from [config.example](config.example) by `install`). It is sourced as shell, so keep it writable only by you.

| Key | Default | Meaning |
|---|---|---|
| `INTERVAL` | `5` | Seconds between checks |
| `LEAVE_CHECKS` | `2` | Consecutive "not in a meeting" checks before turning the Focus off |
| `UI_LANG` | `auto` | Message language: `auto` (follows macOS), `en`, `ja` |
| `ON_SHORTCUT` / `OFF_SHORTCUT` | `Zoom Quiet On` / `Zoom Quiet Off` (Japanese macOS: `Zoom中オン` / `Zoom中オフ`) | Shortcut names to run |
| `ON_CMD` / `OFF_CMD` | empty | Commands to run instead of Shortcuts (used only when both are set) |
| `ACTION_TIMEOUT` | `20` | Seconds before an on/off action is aborted |
| `UDP_MIN` | `1` | Treat Zoom as in a meeting when it has at least this many UDP sockets (`0` disables) |

Apply changes with `launchctl kickstart -k gui/$(id -u)/com.github.zoom-quiet`.

## Caveats

- A notification may slip through in the first few seconds after joining (check interval).
- If the Focus has *Share across devices* on, your iPhone will follow. Turn it off to keep it Mac-only.
- Detection relies on Zoom's internal process names, which a Zoom update could change. `zoom-quiet status` shows what is detected.
- macOS does not expose Focus state, so if you turned `Zoom Quiet` on by hand before a meeting, it will still be turned off when you leave.
- `install` records the script's path in the launch agent. If you move the folder, run `install` again.
- Tested by the author on macOS 26.6 (Apple Silicon) with a real Zoom meeting for detection. Reports from other setups are welcome in Issues.

## Uninstall

```bash
~/zoom-quiet/bin/zoom-quiet uninstall
```

Delete the Focus and the two Shortcuts by hand.

## Videos

- 30-second intro (English): https://chaenmasahiro0425.github.io/zoom-quiet/en/
- 54-second explainer (Japanese captions): https://chaenmasahiro0425.github.io/zoom-quiet/

## License

[MIT](LICENSE)

Zoom is a trademark of Zoom Video Communications, Inc. Claude is a trademark of Anthropic. This is an unofficial tool not affiliated with either company.
