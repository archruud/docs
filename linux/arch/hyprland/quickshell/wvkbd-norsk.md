# Quickshell: virtual keyboard (wvkbd-norsk, optional bar add-on)

*English (primary) — [norsk versjon tilgjengelig](wvkbd-norsk-no.md)*

Full Norwegian on-screen keyboard (æ, ø, å on the correct keys) for
`wvkbd`, the virtual keyboard for wlroots-based Wayland compositors.
The toggle button appears in [bar](quickshell-bar.md) if this is
installed.

The layout itself, the build process, and source files live in the
dedicated repo [archruud/wvkbd-norsk](https://github.com/archruud/wvkbd-norsk)
— this page covers installation and the `hyprland.lua` setup.

Want to make a layout for a language other than Norwegian (Swedish,
Danish, German, etc)? See
[wvkbd-eget-layout.md](wvkbd-eget-layout.md) — a standalone recipe,
not dependent on the rest of this page.

## Installation

```bash
sudo pacman -S --needed wayland libxkbcommon pango cairo scdoc base-devel
```

> The package is called `libxkbcommon`, **not** `xkbcommon` — the
> latter is just the internal `pkg-config` module name used during
> the build.

```bash
mkdir -p ~/Prosjekter/Norsk-tastatur
cd ~/Prosjekter/Norsk-tastatur
git clone https://github.com/jjsullivan5196/wvkbd.git .

git clone https://github.com/archruud/wvkbd-norsk.git /tmp/wvkbd-norsk
cp /tmp/wvkbd-norsk/*.h .

make LAYOUT=norsk
sudo make LAYOUT=norsk install
```

Installs a binary called `wvkbd-norsk` into `/usr/local/bin/`. The
source folder (`~/Prosjekter/Norsk-tastatur/`) can be deleted
afterwards.

## Test directly

```bash
wvkbd-norsk -L 320   # -L = height in pixels, adjust to taste
```

## hyprland.lua

**Autostart**, inside the `hl.on("hyprland.start", function() ... end)`
block, alongside your other autostart lines:

```lua
hl.on("hyprland.start", function()
    -- ... your other autostart lines ...
    hl.exec_cmd("wvkbd-norsk -L 320 --hidden")
end)
```

**Keybind** to show/hide, as its own standalone line among your
other `hl.bind(...)` lines:

```lua
hl.bind(mainMod .. " + T", hl.dsp.exec_cmd("pkill --signal SIGRTMIN wvkbd-norsk"))
```

**Layer rule** (optional — slide up from the bottom instead of just
appearing instantly), as its own standalone block alongside your
other `hl.window_rule`/`hl.layer_rule` entries:

```lua
hl.layer_rule({
    match = { namespace = "wvkbd" },
    animation = "slide",
})
```

## Removing the keyboard button from bar

Install `bar` without selecting `wvkbd-norsk` in
[`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
— then the button is never written into `Bar.qml` in the first
place. Already installed with the keyboard and want to remove the
button afterwards? Remove the block between
`// __KEYBOARD_BUTTON_START__` and `// __KEYBOARD_BUTTON_END__` in
`~/.config/quickshell/bar/widgets/Bar.qml`.
