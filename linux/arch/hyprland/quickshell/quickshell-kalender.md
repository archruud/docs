# Quickshell: Google Calendar button (optional bar add-on)

*English (primary) — [norsk versjon tilgjengelig](quickshell-kalender-no.md)*

The clock module in [bar](quickshell-bar.md) can open a real Google
Calendar in a floating, centered window on click — a thin native
frame (via `ice-ssb`) around the actual website, not a separate API
integration.

**Important to know before setting this up:** regardless of which
technical approach is used (webapp frame or a real API integration),
you'll need to log in with your Google username, password, and the
security code from your authenticator app. This is a Google
requirement for any app that reads your calendar data, and can't be
avoided. This approach at least avoids API-key maintenance on your
end.

## ice-ssb, not webapp-manager — worth knowing why

Both tools exist (`ice-ssb` on the AUR, `webapp-manager` in the
official Arch repo) and share the same underlying folder structure
(`~/.local/share/ice/`). The difference that actually matters here:

- **`ice-ssb`** uses **the exact name you type in** as the profile
  ID/window class — type `GoogleKalender`, get a folder named exactly
  `GoogleKalender`. No random suffix.
- **`webapp-manager`** automatically appends 4 random digits to the
  name **every time** you create the app (e.g. `GoogleKalender3444`
  once, `kalender6845` the next time on reinstall) — confirmed through
  repeated testing.

Since `ice-ssb` gives a **predictable** name, we skip the whole step
of hunting down a generated ID afterward — you decide the name up
front, and use that same name consistently everywhere. That's why
`ice-ssb` is the recommendation here, not `webapp-manager`.

## Procedure

Use **the exact same name** (`GoogleKalender` in the examples below —
change it if you like, but stay consistent) in every step.

### 1. Install ice-ssb

```bash
yay -S ice-ssb
```
(or `paru -S ice-ssb`)

### 2. Get an icon

```bash
rsvg-convert -w 64 -h 64 google-calendar.svg -o ~/.local/share/ice/icons/GoogleKalender.png
```

Placing it in `~/.local/share/ice/icons/` — ice-ssb's own icon
folder — means it should be readily available in ice-ssb's file
picker without having to search the web for one.

### 3. Create the webapp in ice-ssb

Launch `ice-ssb`, click `+`, fill in:

| Field | Value |
|---|---|
| Name | `GoogleKalender` |
| Address | `https://calendar.google.com/calendar/u/0/r?pli=1` |
| Icon | the PNG from step 2 |
| Browser | Firefox |

Click OK, close `ice-ssb`.

Confirm the profile actually got exactly this name:
```bash
ls ~/.local/share/ice/firefox/
```
You should see a folder named exactly `GoogleKalender` — no digits
added.

### 4. Set the name in Bar.qml

Open `~/.config/quickshell/bar/widgets/Bar.qml`, search for
`calendarProc` (around line 243), and insert the name in **all three
places** on the same line (`--class`, `--name`, and the
`--profile` path) — note: **no `WebApp-` prefix** with `ice-ssb`:

```qml
command: ["sh", "-c", "XAPP_FORCE_GTKWINDOW_ICON=\"$HOME/Nedlastinger/google-calendar64.png\" firefox --class GoogleKalender --name GoogleKalender --profile $HOME/.local/share/ice/firefox/GoogleKalender --no-remote \"http://calendar.google.com\""]
```

### 5. hyprland.lua — windowrule for the blue border

Add as a **separate, standalone** `hl.window_rule({...})` block,
alongside your other windowrules (nmgui, blueman, pavucontrol) — not
inside the autostart block. Note again: no `WebApp-` prefix in the
`match` pattern:

```lua
hl.window_rule({
    name  = "float-googlekalender",
    match = { class = "^(GoogleKalender)$" },
    float = true,
    size  = "1000 750",
    center = true,
    animation = "slide",
    border_color = "rgba(2e92dbff)",
})
```

## Skipping this

Set `command: ["true"]` instead of the whole `sh -c` command on the
`calendarProc` line in `Bar.qml`. The clock click then does nothing —
time and date-hover still work normally, no `hyprland.lua` change
needed since the windowrule (step 5) isn't required.

## Automated version

[`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
does steps 1, 2, and 4 automatically, using `GoogleKalender` as the
pre-decided name — it only waits for you to complete step 3 manually
(since `ice-ssb` has no command-line mode), and prints the finished
windowrule text for step 5 at the end. See the
[overview](quickshell-oversikt.md).
