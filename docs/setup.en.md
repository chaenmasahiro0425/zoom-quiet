# First-time setup (about 3 minutes)

macOS has no command line for creating a Focus or a Shortcut, so this part is done by hand.
Labels may differ slightly between macOS versions (written for macOS 26).

> Using macOS in Japanese? Follow [setup.md](setup.md) instead — the default names are `Zoom中` / `Zoom中オン` / `Zoom中オフ` there.

## Step 1: Create the "Zoom Quiet" Focus

1. Open **System Settings → Focus**
2. Click **Add Focus… → Custom**
3. Name it **`Zoom Quiet`** and pick any icon
4. Under **Notifications**, choose **Silence notifications from** (not "Allow")
   - "Allow" would silence every app *except* the ones you pick
5. Add the apps to silence:
   - **Claude** (and any other copies of the Claude app you use)
   - Your terminal (Terminal, iTerm2, Ghostty, …) if you also want Claude Code CLI banners silenced
   - Anything else you want quiet during meetings (Slack, Mail, …)
6. Optionally turn off **Share across devices** — otherwise your iPhone follows the Focus too

## Step 2: Create two Shortcuts

Open the **Shortcuts** app and create these two. The names must match exactly.

### `Zoom Quiet On`

1. Click **+** to create a new shortcut
2. Search for the **Set Focus** action and add it
3. Set it to **turn `Zoom Quiet` On** (until turned off)
4. Name the shortcut **`Zoom Quiet On`**

### `Zoom Quiet Off`

1. Create another shortcut with the **Set Focus** action
2. Set it to **turn `Zoom Quiet` Off**
3. Name it **`Zoom Quiet Off`**

## Step 3: Check

```bash
~/zoom-quiet/bin/zoom-quiet doctor   # both Shortcuts should show ✓
~/zoom-quiet/bin/zoom-quiet on       # the Focus icon should appear in the menu bar
~/zoom-quiet/bin/zoom-quiet off      # and disappear again
```

The first `on` / `off` may ask whether the shortcut may change your Focus. Allow it.

Then enable autostart with `zoom-quiet install`.

## Using other names

If you name the Shortcuts differently, set `ON_SHORTCUT` / `OFF_SHORTCUT` in `~/.config/zoom-quiet/config`.
