# 04 — Kitty (Terminal + Bash Config)

*English (primary) — [norsk versjon tilgjengelig](kitty-setup-no.md)*

## What it is

Sets up bash (prompt, history, aliases) and kitty's own config (font, colors, keybinds) in one go.

## What it does

- Installs `eza` (modern `ls` replacement) and `bat` (syntax-highlighted `cat`) if missing
- Backs up and replaces `~/.bashrc`: minimal folder-only prompt, full alias set (package management via `yay`/`pacman`, git shortcuts, `eza`-based `ls`/`l`/`lt`/`lh`, config-editing shortcuts)
- Backs up and replaces `~/.config/kitty/kitty.conf`: JetBrainsMono Nerd Font at size 18, powerline tab bar, 0.95 background opacity, sensible copy/paste and tab keybinds

## Settings it needs to work

- `hyprconf` alias opens `~/.config/hypr/hyprland.lua` (not `hyprland.conf` — unused since the Lua migration)
- `barconf` alias opens the Quickshell bar's `Bar.qml` (replaces the old `wayconf` alias, which pointed at Waybar's config — gone since `02-quickshell-modules`)
- Requires JetBrainsMono Nerd Font to be installed for the icons in `eza`'s output and the tab bar to render correctly (already in `pacman-packages.txt` under fonts)

## Install

```bash
chmod +x install-kitty-bash.sh
./install-kitty-bash.sh
source ~/.bashrc
```

## Bugs fixed from the original script

- `exa` → `eza`: `exa` is an unmaintained, dead project; `eza` is the actively maintained fork and is what's actually in your package list.
- The `bat` install-check block was duplicated (copy-paste error) and never actually checked for `exa`/`eza` in the second copy — removed the duplicate.
- `hyprconf` and `wayconf` aliases pointed at files that no longer exist post-Lua-migration and post-Quickshell — repointed to the real current files.
