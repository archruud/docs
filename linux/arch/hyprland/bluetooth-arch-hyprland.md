# Bluetooth på Arch Linux + Hyprland

Fullstendig oppsett for Bluetooth — både terminal (`bluetoothctl`) og grafisk (Blueman) — på en Arch/Hyprland-maskin.

## Installasjonsscript

Alle stegene under er automatisert i `bluetooth-arch-hyprland.sh`, som ligger i [archruud/scripts](https://github.com/archruud/scripts):

```bash
curl -O https://raw.githubusercontent.com/archruud/scripts/main/arch-hyprland/bluetooth-arch-hyprland.sh
chmod +x bluetooth-arch-hyprland.sh
./bluetooth-arch-hyprland.sh
```

Scriptet er idempotent — trygt å kjøre på nytt hvis noe endres senere. Resten av dokumentet forklarer hva scriptet gjør, steg for steg, og fungerer som referanse ved feilsøking.

## 1. Pakker

```bash
sudo pacman -S bluez bluez-utils blueman
```

| Pakke | Hva den gir |
|---|---|
| `bluez` | Selve Bluetooth-protokollstacken/daemonen |
| `bluez-utils` | `bluetoothctl` (CLI-verktøy) |
| `blueman` | Grafisk manager (`blueman-manager`) + systray-applet (`blueman-applet`) |

Valgfritt, hvis du trenger gamle/utdaterte verktøy som `hcitool`/`hciconfig`:

```bash
sudo pacman -S bluez-deprecated-tools
```

## 2. Lyd over Bluetooth

Hyprland-oppsett kjører normalt **PipeWire**, ikke PulseAudio. Sjekk hva du faktisk har:

```bash
pactl info | grep "Server Name"
```

Hvis PipeWire (vanligst i moderne Arch/Hyprland-oppsett):

```bash
sudo pacman -S pipewire pipewire-pulse wireplumber
```

WirePlumber håndterer Bluetooth-lyd (A2DP m.m.) automatisk — du trenger **ikke** `pulseaudio-bluetooth` med mindre du faktisk kjører ren PulseAudio i stedet for PipeWire.

## 3. Aktivér og start tjenesten

```bash
sudo systemctl enable --now bluetooth.service
systemctl status bluetooth.service
```

> archinstall kan ha satt tjenesten til enabled uten å starte den i denne sesjonen — kjør kommandoen over uansett, den er trygg å kjøre på nytt.

## 4. Sjekk kernelmodul og adapter

```bash
lsmod | grep btusb
rfkill list
```

- Hvis `btusb` ikke vises: adapteren blir ikke gjenkjent av kernel (sjekk hardware/firmware).
- Hvis `rfkill` viser `Soft blocked: yes`: `sudo rfkill unblock bluetooth`

## 5. Auto-power-on ved boot (valgfritt)

I `/etc/bluetooth/main.conf`:

```ini
[Policy]
AutoEnable=true
```

Uten dette må du kjøre `power on` i `bluetoothctl` (eller åpne Blueman) hver gang du starter maskinen.

## 6. Terminal-bruk (bluetoothctl)

```bash
bluetoothctl
power on
agent on
default-agent
scan on
# vent til enheten dukker opp, noter MAC-adressen
pair XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
```

`trust` gjør at enheten kobler til automatisk neste gang uten ny parring.

## 7. Grafisk bruk i Hyprland (Blueman)

Blueman starter **ikke** automatisk i Hyprland slik det gjør i tyngre DE-er (GNOME/KDE). Legg til i `hyprland.conf`:

```ini
exec-once = blueman-applet
```

Da får du systray-ikon (forutsetter at du har en systray i Waybar/eww/osv. — `tray` modulen i Waybar-config).

Full GUI-manager kan alltid åpnes manuelt:

```bash
blueman-manager
```

## 8. Vanlige feil

| Symptom | Fiks |
|---|---|
| "Protocol not available" / A2DP-feil | Restart PipeWire: `systemctl --user restart pipewire pipewire-pulse wireplumber` |
| Adapter vises ikke i Blueman | `rfkill unblock bluetooth`, sjekk `lsmod \| grep btusb` |
| Enhet kobler ikke til automatisk | Sjekk at du kjørte `trust` i `bluetoothctl`, og at `AutoEnable=true` er satt |
| Ingen systray-ikon | Sjekk at `exec-once = blueman-applet` er lagt til, og at Waybar/tray-modul faktisk kjører |

## Kilder

- [ArchWiki – Bluetooth](https://wiki.archlinux.org/title/Bluetooth)
- [ArchWiki – Blueman](https://wiki.archlinux.org/title/Blueman)
