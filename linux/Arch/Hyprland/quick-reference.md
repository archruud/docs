# Quick Reference - Archruud

**Rask oppslag for de vanligste kommandoene**

---

## Pacman / yay

```bash
sudo pacman -Syu        # Oppdater alt
sudo pacman -S pakke    # Installer
sudo pacman -Rs pakke   # Fjern + avhengigheter
pacman -Ss søk          # Søk
yay -S aur-pakke        # AUR
```

## Systemd

```bash
sudo systemctl start/stop/restart/status tjeneste
sudo systemctl enable --now tjeneste
journalctl -u tjeneste -f    # Logger live
```

## Nettverk

```bash
ip addr                 # Vis IP
ping 8.8.8.8            # Test
ss -tuln                # Åpne porter
```

## Docker

```bash
docker ps               # Kjørende containere
docker ps -a            # Alle containere
docker compose up -d    # Start
docker compose down     # Stopp
docker compose logs -f  # Logger
```

## Filer

```bash
find / -name "fil.txt"  # Finn fil
grep -r "ord" mappe/    # Søk i innhold
ls -lah                 # List detaljer
df -h                   # Diskbruk
du -sh mappe/           # Mappestørrelse
```

## WireGuard VPN

```bash
wu    # wg-quick up wg0
wd    # wg-quick down wg0
ws    # wg show
```

## SSH

```bash
ssh archruud@192.168.30.52      # Dokumentserver
scp fil.md archruud@192.168.30.52:/opt/docker/docs/linux/
```

## Proxmox

```bash
pct list                        # LXC containere
qm list                         # VM liste
zpool status                    # ZFS status
zfs list                        # ZFS datasets
nvidia-smi                      # GPU status
```

## Git

```bash
git status
git add . && git commit -m "melding"
git push
git pull
```

---

## Mine tjenester

| Tjeneste | URL |
|----------|-----|
| Dokumentserver (MkDocs) | http://192.168.30.52:3000 |
| Open WebUI / Ollama | http://192.168.30.x:3000 |
| Proxmox | https://192.168.50.10:8006 |
| Portainer | http://192.168.30.52:9000 |

---

*archruud.org | 2026*
