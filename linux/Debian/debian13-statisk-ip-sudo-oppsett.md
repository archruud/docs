# Debian 13 VM - Statisk IP og Sudo-oppsett

**Server:** stremio-plex-server (VM 210)
**Proxmox-node:** pve1
**Dato:** 7. juli 2026
**Bridge/VLAN:** vmbr1, VLAN tag 10

## Bakgrunn

Under installasjon av Debian 13 (Trixie) på VM 210 ble det satt opp uten sudo-pakke og med DHCP-tildelt IP. Denne guiden dokumenterer hvordan sudo-tilgang og statisk IP ble konfigurert i etterkant.

## 1. Proxmox: VLAN-tag på nettverksgrensesnitt

Nettverksgrensesnittet på VM-en ble satt til riktig bridge og VLAN i Proxmox:

```bash
qm set <VMID> -net0 virtio,bridge=vmbr1,tag=10
```

Kan også verifiseres/settes via Proxmox GUI: **VM → Hardware → Network Device** → `Bridge: vmbr1`, `VLAN Tag: 10`.

## 2. Installere sudo og gi bruker tilgang

Sudo var ikke installert i utgangspunktet. Måtte inn som root via `su -` for å installere pakken og legge til bruker i sudo-gruppen.

```bash
# Logg inn som root
su -

# Installer sudo-pakken
apt update
apt install -y sudo

# Legg til bruker i sudo-gruppen
usermod -aG sudo archruud

exit
```

**Viktig:** Man må logge helt ut og inn igjen (ny SSH-sesjon) etter `usermod` for at gruppemedlemskapet skal tre i kraft:

```bash
exit
ssh archruud@<ip>
sudo whoami   # skal svare "root"
```

### Vanlig fallgruve

`usermod -aG sudo <bruker>` kan kjøres uten at selve `sudo`-pakken er installert. Da får man feilmeldingen `-bash: fant ikke kommando sudo` selv om brukeren står i riktig gruppe. Begge steg (pakkeinstallasjon + gruppemedlemskap) må gjøres.

## 3. Fikse terminal-feil fra Kitty

Ved bruk av Kitty-terminal over SSH oppstod feilen:
```
Error opening terminal: xterm-kitty.
```

Løsning (som root eller via sudo):
```bash
apt install -y kitty-terminfo
```

## 4. Statisk IP-konfigurasjon

Serveren bruker `ifupdown` (standard for Debian netinst uten desktop), så konfigurasjon skjer i `/etc/network/interfaces`.

**Finn interfacenavn:**
```bash
ip a
```
Resultat: `ens18` (VirtIO-grensesnitt fra Proxmox)

**Rediger interfaces-filen:**
```bash
sudo nano /etc/network/interfaces
```

**Komplett filinnhold:**
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens18
auto ens18
iface ens18 inet static
    address 192.168.10.100/24
    gateway 192.168.10.1
    dns-nameservers 192.168.10.1
```

**Restart nettverkstjenesten:**
```bash
sudo systemctl restart networking
```

> **Merk:** Dette kan bryte en aktiv SSH-sesjon hvis IP-en endres. Koble på nytt med den nye IP-en.

## 5. Verifisering

```bash
ip a
```
Bekreftet resultat:
```
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    inet 192.168.10.100/24 brd 192.168.10.255 scope global ens18
```

**Test av gateway og internett-tilgang:**
```bash
ping -c 3 192.168.10.1
ping -c 3 1.1.1.1
```

Resultat:
| Mål | Gjennomsnittlig responstid |
|---|---|
| Gateway (192.168.10.1) | 0.635 ms |
| Internett (1.1.1.1) | 2.787 ms |

0% pakketap på begge - nettverksoppsettet fungerer som forventet.

## Sluttresultat

| Parameter | Verdi |
|---|---|
| Hostname | stremio-plex-server |
| IP-adresse | 192.168.10.100/24 |
| Gateway | 192.168.10.1 |
| DNS | 192.168.10.1 |
| VLAN | 10 |
| Bridge | vmbr1 |
| Sudo-tilgang | archruud ✅ |

## Notater om DNS-valg

Kun lokal DNS (192.168.10.1 / AdGuard Home) ble brukt, uten Cloudflare (1.1.1.1) som sekundær. Dette sikrer at:
- AdGuard Home sin reklameblokkering brukes konsekvent
- Lokal navneoppløsning for interne tjenester (f.eks. `*.archruud.org`) fungerer
- UCG Max videresender uansett til oppstrøms DNS ved cache-miss, så ingen funksjonalitet går tapt

Hvis AdGuard går ned og redundans er ønskelig, kan en sekundær DNS legges til:
```
dns-nameservers 192.168.10.1 1.1.1.1
```
