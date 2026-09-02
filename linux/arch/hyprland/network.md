# 18 — Nettverk & Samba Client

## Hva det er

Setter Dolphin opp som en fullverdig nettverksklient: SMB/Windows-deling, NFS, SSH/SFTP, mDNS-oppdagelse (Avahi), mobiltilkobling (MTP/AFC), og hjelpescript for skanning og montering. **Ren klient** — ingen server-tjenester startes eller eksponeres.

## Hva det gjør

- Installerer protokoll-drivere: `smbclient`, `cifs-utils`, `gvfs-smb`, `nfs-utils`, `gvfs-nfs`, `avahi`, `nss-mdns`, `gvfs-mtp`, `gvfs-afc`, `kio-extras`, `kdeconnect` m.fl.
- (AUR) `kio-gdrive` for Google Drive i Dolphin, hvis `yay` finnes
- Setter opp `/etc/samba/smb.conf` som ren klient (ingen `smbd`/`nmbd` server-tjenester)
- Lager mount points: `/mnt/network`, `/mnt/smb`, `/mnt/nfs`, `/mnt/ftp`
- Legger nettverkssnarveier inn i Dolphins sidepanel (`smb:/`, `ftp:/`, `remote:/`)
- Installerer tre hjelpekommandoer i `~/.local/bin/`:
  - `scan-network` — skanner alle definerte VLAN-er (10, 20, 30, 40, 50, 75) + mDNS + NetBIOS
  - `mount-smb //server/share mountpoint [bruker] [passord]` — monterer SMB-share, prøver SMB 3.0 → 2.1 → 2.0
  - `test-protocols` — sjekker at alt er installert og kjører
- Stopper og deaktiverer eventuelle server-tjenester (`smbd`, `nmbd`, `winbind`, `vsftpd`) for å garantere ren klient-modus

## Settings den trenger for å fungere

- **VLAN-numrene i `scan-network`** (10/20/30/40/50/75) er hardkodet til ditt UniFi-oppsett — juster i scriptet hvis VLAN-planen endres
- **Keybinds må inn i `hyprland.lua` manuelt** — scriptet sjekker om de finnes og printer de fire linjene hvis ikke (se under). Skriver aldri til `hyprland.conf` lenger (den filen leses ikke etter Lua-migreringen)

```lua
hl.bind(mainMod .. " + N",         hl.dsp.exec_cmd("dolphin smb:/"))
hl.bind(mainMod .. " + SHIFT + N", hl.dsp.exec_cmd("dolphin remote:/"))
hl.bind(mainMod .. " + CTRL + N",  hl.dsp.exec_cmd("dolphin mtp:/"))
hl.bind(mainMod .. " + ALT + N",   hl.dsp.exec_cmd("~/.local/bin/scan-network"))
```

## Endringer fra originalen

- De to identiske filene (`install-network_script.sh` og `network_script.sh`) er slått sammen til én: `install-network.sh`
- Keybind-delen skriver ikke lenger blindt til `hyprland.conf` — sjekker `hyprland.lua` og viser deg de fire linjene å legge inn selv
- Loggfilnavnet er konsekvent `install-18-network-<dato>.log`

## Installasjon

```bash
chmod +x install-network.sh
./install-network.sh
```

Full logg lagres automatisk i `~/install-18-network-<dato>.log`. Scriptet spør til slutt om du vil kjøre `scan-network` og åpne SMB-browseren i Dolphin med det samme.
