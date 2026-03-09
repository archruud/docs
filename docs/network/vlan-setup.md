# VLAN Nettverk - Arkruud Hjemmenettverk

**For:** Archruud | **Oppdatert:** 2026  
**Utstyr:** UniFi UCG Max, Pro Max 24, Aggregation Switch, U7-Pro-Wall APs

---

## VLAN Oversikt

| VLAN ID | Navn | Subnet | Formål |
|---------|------|--------|--------|
| 1 | Default | 192.168.1.0/24 | Native VLAN |
| 10 | Nordli | 192.168.10.0/24 | Hoved-LAN, klienter |
| 20 | IoT | 192.168.20.0/24 | IoT-enheter, isolert |
| 30 | Servers | 192.168.30.0/24 | Proxmox, NAS, Docker |
| 40 | ARR Media | 192.168.40.0/24 | Media-stack, LXC 411-422 |
| 50 | Admin | 192.168.50.0/24 | Nettverksadmin, iDRAC |
| 75 | Testing | 192.168.75.0/24 | Isolert test-miljø |
| 101 | IPTV | 192.168.101.0/24 | Altibox IPTV |

---

## Viktige IP-adresser

```
UCG Max gateway:        192.168.50.1
Proxmox node 1:        192.168.50.10
Proxmox node 2:        192.168.50.11
Dokumentserver:        192.168.30.52  (port 3000)
Open WebUI/Ollama:     192.168.30.x   (port 3000/11434)
```

---

## UniFi VLAN oppsett

### Opprett VLAN

1. **Settings → Networks → Create New Network**
2. Sett VLAN ID, navn og subnet
3. Konfigurer DHCP hvis ønskelig

### Port Profiles

```
Trunk port:     Tagged med alle VLANs (til uplinks, servere)
Access port:    Untagged med ett VLAN (til klienter)
```

### Firewall regler

```
IoT → Main LAN:     BLOKKERT
IoT → Internet:     TILLATT
Testing → ALT:      BLOKKERT
Admin → ALT:        TILLATT
```

---

## ARR Media Stack (VLAN 40)

LXC Containere med ID 411-422:

| LXC ID | Tjeneste | IP | Port |
|--------|---------|-----|------|
| 411 | Prowlarr | 192.168.40.11 | 9696 |
| 412 | Radarr | 192.168.40.12 | 7878 |
| 413 | Sonarr | 192.168.40.13 | 8989 |
| 414 | Lidarr | 192.168.40.14 | 8686 |
| 415 | Readarr | 192.168.40.15 | 8787 |
| 416 | Bazarr | 192.168.40.16 | 6767 |
| 417 | Tdarr | 192.168.40.17 | 8265 |
| 418 | Unpackerr | 192.168.40.18 | - |
| 419 | Notifiarr | 192.168.40.19 | 5454 |
| 420 | Plex | 192.168.40.20 | 32400 |
| 421 | Overseerr | 192.168.40.21 | 5055 |
| 422 | Caddy | 192.168.40.22 | 80/443 |

---

## Arch Linux VLAN-konfigurasjon

For å koble laptop til spesifikt VLAN:

```bash
# Se nettverksgrensesnitt
ip addr

# Manuell VLAN (midlertidig)
sudo ip link add link eth0 name eth0.30 type vlan id 30
sudo ip addr add 192.168.30.100/24 dev eth0.30
sudo ip link set eth0.30 up

# NetworkManager (permanent)
nmcli connection add type vlan ifname vlan30 dev eth0 id 30
nmcli connection modify vlan30 ipv4.addresses 192.168.30.100/24
nmcli connection modify vlan30 ipv4.method manual
nmcli connection up vlan30
```

---

*Sist oppdatert: 2026 | archruud.org*
