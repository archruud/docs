# NordVPN på Arch Linux (Hyprland)

**For:** Archruud | **Oppdatert:** 2026

---

## Installasjon

```bash
# Via AUR
yay -S nordvpn-bin
```

### Aktiver tjeneste

```bash
sudo systemctl enable --now nordvpnd
sudo usermod -aG nordvpn $USER
# Logg ut og inn igjen!
```

---

## Bruk

```bash
# Logg inn
nordvpn login

# Koble til
nordvpn connect              # Nærmeste server
nordvpn connect Norway       # Spesifikt land
nordvpn connect Oslo         # Spesifikk by

# Status
nordvpn status

# Koble fra
nordvpn disconnect

# List servere
nordvpn servers
nordvpn servers --country Norway
```

---

## Innstillinger

```bash
# Vis innstillinger
nordvpn settings

# Killswitch (blokkerer internett hvis VPN kobles fra)
nordvpn set killswitch on
nordvpn set killswitch off

# Auto-connect
nordvpn set autoconnect on

# Teknologi (NordLynx = WireGuard, raskest)
nordvpn set technology NordLynx
```

---

## Fjerning

```bash
# Koble fra og logg ut
nordvpn disconnect
nordvpn logout

# Stopp tjeneste
sudo systemctl stop nordvpnd
sudo systemctl disable nordvpnd

# Fjern pakke
yay -Rns nordvpn-bin

# Fjern konfig-filer
rm -rf ~/.config/nordvpn
sudo rm -rf /var/lib/nordvpn
```

---

## Feilsøking

```bash
# Tjeneste kjører ikke
sudo systemctl status nordvpnd
sudo systemctl start nordvpnd

# Permission denied
groups $USER    # Sjekk at nordvpn er i listen
# Logg ut og inn igjen hvis ikke

# Sjekk logger
journalctl -u nordvpnd -n 50
```

---

*Sist oppdatert: 2026 | archruud.org*
