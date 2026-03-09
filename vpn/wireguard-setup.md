# WireGuard VPN - Oppsett på UCG Max

**For:** Archruud | **Oppdatert:** 2026  
**Router:** UniFi UCG Max | **Subnet:** 10.50.50.0/24

---

## Oversikt

WireGuard VPN gir deg tilgang til hele hjemmenettverket fra hvor som helst. Oppsatt på UCG Max-ruteren.

```
Klient (mobil/laptop)
    ↓ WireGuard tunnel
UCG Max (10.50.50.1)
    ↓ tilgang til alle VLANs
VLAN 10 (192.168.10.x) - Nordli
VLAN 20 (192.168.20.x) - IoT
VLAN 30 (192.168.30.x) - Servere
VLAN 40 (192.168.40.x) - ARR Media
VLAN 50 (192.168.50.x) - Admin
```

---

## Oppsett på UCG Max

### I UniFi Network GUI:

1. Gå til **Settings → Teleport & VPN → Site-to-Site VPN**
2. Velg **WireGuard**
3. Konfigurer:
   - **VPN Subnet:** `10.50.50.0/24`
   - **Listen Port:** `51820`

---

## Klient på Arch Linux

### Installer WireGuard

```bash
sudo pacman -S wireguard-tools
```

### Konfigurasjonsfil

Lag `/etc/wireguard/wg0.conf`:

```ini
[Interface]
PrivateKey = DIN_PRIVATE_KEY
Address = 10.50.50.2/32
# DNS = 192.168.50.1    # Kommenter ut for å unngå DNS-konflikter

[Peer]
PublicKey = UCG_MAX_PUBLIC_KEY
Endpoint = DIN_EKSTERN_IP:51820
AllowedIPs = 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24, 192.168.40.0/24, 192.168.50.0/24, 10.50.50.0/24
PersistentKeepalive = 25
```

### Start/stopp

```bash
sudo wg-quick up wg0
sudo wg-quick down wg0
sudo wg show            # Vis status
```

### Aliaser i ~/.bashrc

```bash
alias wu='sudo wg-quick up wg0'
alias wd='sudo wg-quick down wg0'
alias ws='sudo wg show'
alias wl='watch -n 1 sudo wg show'
```

### Sudoers (ingen passord for WireGuard)

```bash
sudo visudo
# Legg til:
archruud ALL=(ALL) NOPASSWD: /usr/bin/wg-quick up wg0, /usr/bin/wg-quick down wg0, /usr/bin/wg show
```

---

## Klient på Android

1. Last ned **WireGuard** fra Play Store
2. Trykk `+` → **Create from QR code** eller **Import from file**
3. Eksporter konfig fra UCG Max som QR eller fil
4. Aktiver tunnelen

---

## Feilsøking

```bash
# Sjekk om tilkoblet
sudo wg show

# Test tilkobling
ping 192.168.30.52      # Dokumentserver

# DNS-konflikt på Arch
# Kommenter ut DNS= linjen i wg0.conf
sudo wg-quick down wg0
# Rediger /etc/wireguard/wg0.conf → kommenter DNS
sudo wg-quick up wg0
```

---

*Sist oppdatert: 2026 | archruud.org*
