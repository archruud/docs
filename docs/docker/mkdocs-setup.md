# MkDocs-Material Dokumentserver

**For:** Archruud | **Oppdatert:** 2026  
**Server:** 192.168.30.52 | **Port:** 3000  
**Dokument-mappe:** `/opt/docker/docs/`

---

## Arkitektur

```
/opt/docker/
├── docs/               ← KUN markdown-filer her
│   ├── linux/
│   ├── bash/
│   ├── hyprland/
│   ├── ai/
│   ├── vpn/
│   ├── network/
│   ├── docker/
│   └── proxmox/
├── mkdocs/             ← MkDocs konfig
│   ├── docker-compose.yml
│   └── mkdocs.yml
└── filebrowser/        ← For å laste opp/ned filer via web
```

---

## Docker Compose

Fil: `/opt/docker/mkdocs/docker-compose.yml`

```yaml
version: '3'

services:
  mkdocs:
    image: squidfunk/mkdocs-material
    container_name: mkdocs
    ports:
      - "3000:8000"
    volumes:
      - /opt/docker/docs:/docs
      - /opt/docker/mkdocs/mkdocs.yml:/docs/mkdocs.yml
    restart: unless-stopped
```

---

## mkdocs.yml

Fil: `/opt/docker/mkdocs/mkdocs.yml`

```yaml
site_name: Archruud Dokumentasjon
site_url: http://192.168.30.52:3000

theme:
  name: material
  palette:
    scheme: slate
    primary: deep purple
    accent: purple
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - search.highlight
    - content.code.copy

plugins:
  - search:
      lang: 'no'

nav:
  - Hjem: index.md
  - Linux:
    - Arch Kommandoer: linux/arch-kommandoer.md
    - Vim Guide: linux/vim-guide.md
    - Søking i Linux: linux/soking-guide.md
    - Tips og Aliaser: linux/arch-tips-aliases.md
    - SCP Guide: linux/scp-guide.md
    - Bruker og Docker: linux/bruker-rettigheter.md
  - Bash:
    - Bash Scripting: bash/bash-scripting-guide.md
  - Hyprland:
    - Tmux Dropdown: hyprland/tmux-dropdown.md
    - hdrop: hyprland/hdrop-guide.md
  - AI:
    - Ollama Terminal: ai/ollama-terminal.md
    - Open WebUI Proxmox: ai/openwebui-proxmox-lxc.md
  - VPN:
    - WireGuard: vpn/wireguard-setup.md
    - NordVPN Arch: vpn/nordvpn-arch.md
  - Nettverk:
    - VLAN Oppsett: network/vlan-setup.md
  - Docker:
    - MkDocs Oppsett: docker/mkdocs-setup.md
  - Proxmox:
    - ZFS Proxmox 9: proxmox/zfs-proxmox9.md
    - NVIDIA Drivere: proxmox/nvidia-guide.md
```

---

## Start og stopp

```bash
cd /opt/docker/mkdocs

# Start
docker compose up -d

# Stopp
docker compose down

# Restart
docker compose restart

# Se logger
docker compose logs -f
```

---

## Legge til dokumenter

Bare dropp en `.md` fil i riktig undermappe under `/opt/docker/docs/`:

```bash
# Fra laptop via SCP
scp guide.md archruud@192.168.30.52:/opt/docker/docs/linux/

# Fra server direkte
nano /opt/docker/docs/linux/ny-guide.md
```

MkDocs oppdateres automatisk (live reload) uten restart.

Husk å legge til filen i `nav:` seksjonen i `mkdocs.yml`!

---

## FileBrowser (valgfritt)

For å laste opp filer via web-grensesnitt:

```yaml
# Legg til i docker-compose.yml
  filebrowser:
    image: filebrowser/filebrowser
    container_name: filebrowser
    ports:
      - "8080:80"
    volumes:
      - /opt/docker/docs:/srv
    restart: unless-stopped
```

Tilgang: `http://192.168.30.52:8080`

---

*Sist oppdatert: 2026 | archruud.org*
