# KVM/QEMU — Testmiljø

*[English (primary)](kvm-qemu-setup.md) — norsk versjon*

*Del av `additional-programs/` — ikke del av den nummererte grunninstallasjonen. Hentes frem når du vil ha et isolert sted å teste noe før du bestemmer deg for om det hører hjemme på det ekte systemet.*

## Hva det er

KVM/QEMU + virt-manager, satt opp rent for **engangs-testing** — prøve en distro, en config, eller et program før du forplikter deg til det på ekte maskinvare. Ikke ment å kjøre permanent.

## Nettverk — den viktige delen

VM-en får sin **egen interne IP-serie**, helt separat fra LAN-et ditt:

- **NAT via `virbr0`** (styrt av `dnsmasq`), subnet `192.168.122.0/24`
- All utgående trafikk går via archminis egen tilkobling — VM-en er usynlig for resten av nettverket (VLAN 50 og videre ser den aldri, ingen IP-reservasjon nødvendig på UniFi-siden)
- **Ingen bridge-konfigurasjon** — bevisst enklere enn en ekte bridge ville vært, siden VM-en aldri trenger å være nåbar fra andre enheter

Samme oppsett som ble bekreftet fungerende på Medion Erazer (feb 2026) — gjenbrukt her uendret for archmini.

## Hva det gjør

- Installerer `qemu-desktop`, `libvirt`, `virt-manager`, `virt-viewer`, `dnsmasq`, `iptables-nft`, `edk2-ovmf`, `bridge-utils`, `dmidecode`
- Aktiverer og starter `libvirtd.socket`
- Legger brukeren din til i `libvirt`- og `kvm`-gruppene
- Starter og autostarter standard NAT-nettverket (`virbr0`)
- Lager `~/.local/share/libvirt/{images,iso}` for VM-disker og installer-ISO-er

## Innstillinger den trenger for å fungere

- **Logg ut og inn igjen (eller reboot) etter installasjon** — gruppemedlemskap (`libvirt`, `kvm`) trer først i kraft ved neste innlogging.
- Legg installer-ISO-er i `~/.local/share/libvirt/iso/` — Virt-Manager sin filvelger finner dem der automatisk.

## Installasjon

```bash
chmod +x install-kvm-qemu.sh
./install-kvm-qemu.sh
```

Deretter: `virt-manager` for å åpne GUI-et og lage en VM.

## Fjerne det etterpå

Dette er ment å være midlertidig — ren fjerning er printet på slutten av scriptet:

```bash
virsh undefine <vm-navn> --remove-all-storage   # fjern VM-en + disken
sudo pacman -Rns qemu-desktop libvirt virt-manager virt-viewer dnsmasq edk2-ovmf
```
