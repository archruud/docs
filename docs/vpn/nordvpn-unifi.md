# NordVPN WireGuard (NordLynx) for UniFi Gateway Max

**For:** Archruud | **Oppdatert:** 2026

---

## Forutsetninger

- NordVPN-konto med aktiv abonnement
- Access Token (IKKE brukernavn/passord — NordVPN sluttet med det!)

### Hent Access Token

1. Gå til https://my.nordaccount.com
2. **Services → NordVPN → Access Token**
3. Generer ny token

### Hent NordLynx Private Key

```bash
# Installer NordVPN CLI midlertidig
yay -S nordvpn-bin
nordvpn login --token DIN_ACCESS_TOKEN
nordvpn set technology NordLynx
nordvpn connect Norway

# Hent private key
wg show nordlynx
# Output: private key = XXXXXXXX...

nordvpn disconnect
```

---

## Finn norsk server-informasjon

```bash
# Hent liste over norske WireGuard-servere
curl "https://api.nordvpn.com/v1/servers?filters\[servers_technologies\]\[identifier\]=wireguard_udp&filters\[country_id\]=163&limit=5" | python3 -m json.tool
```

Fra output, noter:
- `station` = server IP-adresse
- `public_key` = server public key (i technologies/metadata)

---

## UniFi Gateway konfigurasjon

### I UniFi Network GUI:

1. **Settings → VPN → WireGuard**
2. **Create WireGuard VPN**

**Interface:**
```
Name:        nordvpn-norway
Private Key: DIN_NORDLYNX_PRIVATE_KEY
Address:     10.5.0.2/32
DNS:         103.86.96.100   (NordVPN DNS)
MTU:         1420
```

**Peer:**
```
Public Key:  SERVER_PUBLIC_KEY
Endpoint:    SERVER_IP:51820
AllowedIPs:  0.0.0.0/0   (all trafikk via VPN)
Keepalive:   25
```

---

## Traffic Routes (hvilke enheter bruker VPN)

### Alt VLAN 40 via VPN:

```
Policy:     ARR-via-VPN
Source:     192.168.40.0/24
Destination: 0.0.0.0/0
Interface:  nordvpn-norway
```

### Spesifikke enheter:

```
Policy:     device-via-vpn
Source:     192.168.10.50/32
Destination: 0.0.0.0/0
Interface:  nordvpn-norway
```

---

## Testing

```bash
# Fra enhet på VPN-VLAN
curl ifconfig.me           # Skal vise norsk/NordVPN IP
curl ipinfo.io/country     # Skal returnere NO

# DNS-test
nslookup google.com 103.86.96.100
```

---

## Feilsøking

```bash
# Hvis VPN ikke kobler til
# → Sjekk at NordLynx private key er riktig
# → Prøv annen norsk server
# → Sjekk port 51820 er ikke blokkert av ISP

# DNS-lekkasje
curl https://dnsleaktest.com/   # Test i browser
```

---

*Sist oppdatert: 2026 | archruud.org*
