# ZFS og Proxmox 9 - Oppgradering og Feilsøking

**For:** Archruud | **Oppdatert:** 2026  
**System:** Proxmox VE 9.x, Dell PowerEdge T620/T420/T330

---

## Problemet

Etter oppgradering til Proxmox 9 kjøres feil kernel (Debian-kernel), noe som hindrer ZFS-modulene fra å laste:

```
Error: could not activate storage 'sas-zfs-pool'
zfs error: Try running 'modprobe zfs' as root
```

**Årsak:** Systemet booter Debian-kernel (`6.12.x-deb13`) i stedet for Proxmox-kernel (`6.8.x-pve`). ZFS-modulene er kompilert for Proxmox-kernelen.

---

## Diagnose

```bash
# Sjekk hvilken kernel som kjører
uname -r
# Output: 6.12.69+deb13-amd64  ← FEIL! Skal være 6.8.x-pve

# List installerte kernels
dpkg -l | grep proxmox-kernel

# Sjekk boot-konfig
proxmox-boot-tool status
```

---

## Løsning: Pin Proxmox-kernel

```bash
# Finn tilgjengelig Proxmox-kernel
dpkg -l | grep proxmox-kernel
# F.eks.: proxmox-kernel-6.8.12-15-pve

# Pin kernel
proxmox-boot-tool kernel pin 6.8.12-15-pve

# Oppdater bootloader
proxmox-boot-tool refresh

# Reboot
reboot
```

### Etter reboot

```bash
# Verifiser riktig kernel
uname -r
# Output: 6.8.12-15-pve  ← RIKTIG!

# Sjekk ZFS
zpool status
zfs list

# Sjekk containere
pct list
```

---

## ZFS Pool administrasjon

```bash
# Status alle pools
zpool status

# Detaljert status
zpool status -v

# Disk-bruk
zfs list

# Import pool (etter reboot)
zpool import sas-zfs-pool
zpool import nvme-zfs-pool
zpool import ssd-zfs-pool

# Auto-import (settes i /etc/zfs/zpool.cache)
zpool set cachefile=/etc/zfs/zpool.cache poolnavn
```

---

## Forebygging

```bash
# Sjekk alltid kernel etter oppdatering
apt upgrade
proxmox-boot-tool refresh
# Restart og verifiser uname -r
```

---

## ZFS Snapshot

```bash
# Lag snapshot
zfs snapshot poolnavn/dataset@snapshot-navn

# List snapshots
zfs list -t snapshot

# Rollback
zfs rollback poolnavn/dataset@snapshot-navn

# Slett snapshot
zfs destroy poolnavn/dataset@snapshot-navn
```

---

*Sist oppdatert: 2026 | archruud.org*
