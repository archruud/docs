# Archruud Dokumentasjon

> Teknisk dokumentasjon for Arch Linux, Proxmox/Debian, Nettverk, AI og mer.
> Strukturert etter **operativsystem**, ikke programvare — når du
> feilsøker er det OS-en (og versjonen) som avgjør hva som gjelder.

---

## Struktur

```
linux/
├── arch/                    ← Arch Linux (Hyprland, kommandoer, tips)
│   └── hyprland/             ← alt Hyprland-spesifikt
├── debian/
│   ├── 13/                   ← Debian 13 (Trixie)
│   │   ├── proxmox/9.1.5/    ← Proxmox VE på Debian 13
│   │   └── truenas/          ← (klar for NAS-dokumentasjon)
│   └── 12/
│       └── proxmox/8/        ← legacy Proxmox VE 8 / Debian 12

ai/        AI-verktøy (Ollama, Open WebUI) — programvarelag, ikke OS-spesifikt
vpn/       WireGuard, NordVPN — dels UniFi-appliance, ikke Linux
network/   VLAN/UniFi-nettverk
docker/    Selve dokumentserveren
bash/      Generell scripting
```

---

## Maskinvare

| Enhet | Spesifikasjoner |
|-------|----------------|
| Dell Pro 16 | Arch Linux · Hyprland · Intel iGPU · ekstern Lenovo Legion 27Q-10 |
| Medion Erazer X10 | Arch Linux · Intel Arc A730M · 64GB DDR5 |
| Dell PowerEdge T620 | Proxmox VE 9.1.5 · 2x NVIDIA A2 + Tesla T4 · ZFS |
| Dell PowerEdge T420 | Proxmox |
| MacBook Pro M4 | macOS |

---

*archruud.org · Oppdatert 27.08.2026*
