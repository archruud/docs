# Feilsøkingsguide: Kernel-oppdatering knekker NVIDIA GPU (Proxmox)

**For:** pve1 (Dell PowerEdge T620) | **Oppdatert:** juli 2026
**GPUer:** 2x NVIDIA A2 + 1x Tesla T4
**Driver:** NVIDIA 595.71.05 (DKMS)

---

## Symptomet

Du kjører `apt update && apt upgrade`, reboot, og plutselig:

```bash
nvidia-smi
# NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
```

Container(e) med GPU passthrough nekter å starte:

```
Error: Device /dev/nvidia-caps/nvidia-cap2 ...
```

**Årsak:** `apt upgrade` installerte en ny Proxmox-kernel. NVIDIA-driveren er DKMS-basert og bygges *per kernel-versjon* — når en ny kernel dukker opp uten at DKMS har bygget modulen for akkurat den, forsvinner GPU-en helt til modulen er bygget.

---

## Rask diagnose (kjør disse to først, alltid)

```bash
uname -r
dkms status
```

Sammenlign: er kernel-versjonen fra `uname -r` **med** i `dkms status`-lista som `installed`?

- **Nei** → dette er problemet, gå til fiksen under.
- **Ja** → noe annet er galt, sjekk `dmesg | tail -50` og `modprobe nvidia` for faktisk feilmelding.

---

## Fiksen (host)

### 1. Installer headers for kjørende kernel

```bash
apt install -y proxmox-headers-$(uname -r)
```

> Merk: pakken heter `proxmox-headers-X`, ikke `pve-headers-X` — apt foreslår riktig navn automatisk hvis du skriver feil.

### 2. Bygg NVIDIA-modulen for denne kernelen

```bash
dkms install -m nvidia -v 595.71.05 -k $(uname -r)
```

Dette signerer og bygger `nvidia.ko`, `nvidia-uvm.ko`, `nvidia-modeset.ko`, `nvidia-drm.ko`, `nvidia-peermem.ko` og installerer dem i `/lib/modules/$(uname -r)/updates/dkms/`.

### 3. Last inn og verifiser

```bash
modprobe nvidia
nvidia-smi
nvidia-smi -L    # bekreft at alle 3 GPUer (2x A2 + T4) vises
```

### Hvis DKMS-bygget feiler (mangler kompilator)

```bash
apt install -y build-essential
dkms install -m nvidia -v 595.71.05 -k $(uname -r)
```

---

## Etter host er fikset — sjekk containerne

Device-nodene (`/dev/nvidia*`, `/dev/nvidia-caps/*`) opprettes automatisk når modulen lastes. Start containerne på nytt:

```bash
pct start 100   # OllamaAI
pct start 101   # hermes
pct start 300   # helseweb/paperless
# ...alle andre med GPU passthrough
```

### Test hver container

```bash
pct enter <CTID>
nvidia-smi
```

**Fungerer det?** Bra, gå videre til neste.

**"Driver/library version mismatch"?** Containeren har en gammel driver installert inni seg som ikke matcher host lenger. Fiks:

```bash
# Inni containeren
cd /tmp
wget https://us.download.nvidia.com/tesla/595.71.05/NVIDIA-Linux-x86_64-595.71.05.run
chmod +x NVIDIA-Linux-x86_64-595.71.05.run
./NVIDIA-Linux-x86_64-595.71.05.run --no-kernel-module --silent
nvidia-smi
```

> `--no-kernel-module` er kritisk — containeren skal kun ha userspace-bibliotekene, kernel-modulen kommer fra host.

---

## Fallback: pin tilbake til fungerende kernel

Hvis DKMS-bygget krangler og du trenger GPU *nå*:

```bash
proxmox-boot-tool status                        # se hvilke kernels som er tilgjengelige
proxmox-boot-tool kernel pin <fungerende-versjon>
proxmox-boot-tool refresh
reboot
```

Etter reboot, verifiser:

```bash
uname -r         # skal vise pinnet versjon
nvidia-smi       # skal fungere med en gang (DKMS-modul finnes allerede)
zpool status     # sjekk at ZFS pools også er friske
```

---

## Sjekkliste — kjør i denne rekkefølgen ved neste kernel-relaterte GPU-problem

1. `uname -r` — hvilken kernel kjører?
2. `dkms status` — er modulen bygget for denne kernelen?
3. Nei → `apt install -y proxmox-headers-$(uname -r)`
4. `dkms install -m nvidia -v 595.71.05 -k $(uname -r)`
5. `modprobe nvidia && nvidia-smi`
6. Start containerne én etter én, test `nvidia-smi` i hver
7. Mismatch i CT → oppdater driver inni CT med `.run --no-kernel-module --silent`

---

## Forebygging (fremtidig forbedring — ikke satt opp ennå)

Et apt-hook som automatisk kjører `dkms autoinstall` rett etter `apt upgrade`, *før* reboot, ville fanget dette proaktivt. Kan settes opp i `/etc/apt/apt.conf.d/` som en `Post-Invoke`-hook. Si fra hvis du vil ha dette — er en engangsjobb på ti minutter.

---

*Sist oppdatert: juli 2026 | archruud.org*
