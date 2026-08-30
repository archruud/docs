# Quickshell: virtuelt tastatur (wvkbd-norsk, valgfritt tillegg til bar)

*Norsk — [English primary version](wvkbd-norsk.md)*

Fullt norsk on-skjerm-tastatur (æ, ø, å på riktige taster) for
`wvkbd`, det virtuelle tastaturet for wlroots-baserte Wayland-
compositors. Toggle-knappen dukker opp i [bar](quickshell-bar.md)
hvis dette er installert.

Selve layoutet, byggeprosessen og kildefilene ligger i det egne
repoet [archruud/wvkbd-norsk](https://github.com/archruud/wvkbd-norsk)
— denne siden dekker installasjon og `hyprland.lua`-oppsettet.

Ønsker du å lage et layout for et annet språk enn norsk (svensk,
dansk, tysk osv.)? Se
[wvkbd-eget-layout.md](wvkbd-eget-layout.md) — en frittstående
oppskrift, ikke avhengig av resten av denne siden.

## Installasjon

```bash
sudo pacman -S --needed wayland libxkbcommon pango cairo scdoc base-devel
```

> Pakken heter `libxkbcommon`, **ikke** `xkbcommon` — det siste er
> kun `pkg-config`-modulnavnet internt i byggeprosessen.

```bash
mkdir -p ~/Prosjekter/Norsk-tastatur
cd ~/Prosjekter/Norsk-tastatur
git clone https://github.com/jjsullivan5196/wvkbd.git .

git clone https://github.com/archruud/wvkbd-norsk.git /tmp/wvkbd-norsk
cp /tmp/wvkbd-norsk/*.h .

make LAYOUT=norsk
sudo make LAYOUT=norsk install
```

Installerer en binær kalt `wvkbd-norsk` i `/usr/local/bin/`. Kilde-
mappen (`~/Prosjekter/Norsk-tastatur/`) kan slettes etterpå.

## Test direkte

```bash
wvkbd-norsk -L 320   # -L = høyde i piksler, juster til din smak
```

## hyprland.lua

**Autostart**, inni `hl.on("hyprland.start", function() ... end)`-
blokken, sammen med dine andre autostart-linjer:

```lua
hl.on("hyprland.start", function()
    -- ... dine andre autostart-linjer ...
    hl.exec_cmd("wvkbd-norsk -L 320 --hidden")
end)
```

**Keybind** for å vise/skjule, som egen frittstående linje blant dine
andre `hl.bind(...)`-linjer:

```lua
hl.bind(mainMod .. " + T", hl.dsp.exec_cmd("pkill --signal SIGRTMIN wvkbd-norsk"))
```

**Layer rule** (valgfritt — sprett opp fra bunnen i stedet for å bare
dukke opp momentant), som egen frittstående blokk sammen med dine
andre `hl.window_rule`/`hl.layer_rule`-oppføringer:

```lua
hl.layer_rule({
    match = { namespace = "wvkbd" },
    animation = "slide",
})
```

## Fjerne tastatur-knappen fra bar

Installer `bar` uten å velge `wvkbd-norsk` i
[`install-quickshell.sh`](https://github.com/archruud/scripts/blob/main/arch-hyprland/install-quickshell.sh)
— da skrives knappen aldri inn i `Bar.qml` i utgangspunktet. Har du
allerede installert med tastaturet og vil fjerne knappen i etterkant,
fjern blokken mellom `// __KEYBOARD_BUTTON_START__` og
`// __KEYBOARD_BUTTON_END__` i `~/.config/quickshell/bar/widgets/Bar.qml`.
