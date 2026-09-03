# KVM/QEMU — Testing Environment

*English (primary) — [norsk versjon tilgjengelig](kvm-qemu-setup-no.md)*

*Part of `additional-programs/` — not part of the numbered base install. Pull this in whenever you want an isolated place to test something before deciding if it belongs on the real system.*

## What it is

KVM/QEMU + virt-manager, set up purely for **throwaway testing** — trying out a distro, a config, or a piece of software before committing to it on real hardware. Not meant to run persistently.

## Network — the important part

The VM gets its **own internal IP range**, completely separate from your LAN:

- **NAT via `virbr0`** (managed by `dnsmasq`), subnet `192.168.122.0/24`
- All outbound traffic routes through archmini's own connection — the VM is invisible to the rest of your network (VLAN 50 and beyond never see it, no IP reservation needed on the UniFi side)
- **No bridge configuration** — deliberately simpler than a real bridge would be, since the VM never needs to be reachable from other devices

This is the same setup verified working on Medion Erazer (Feb 2026) — reused here as-is for archmini.

## What it does

- Installs `qemu-desktop`, `libvirt`, `virt-manager`, `virt-viewer`, `dnsmasq`, `iptables-nft`, `edk2-ovmf`, `bridge-utils`, `dmidecode`
- Enables and starts `libvirtd.socket`
- Adds your user to the `libvirt` and `kvm` groups
- Starts and autostarts the default NAT network (`virbr0`)
- Creates `~/.local/share/libvirt/{images,iso}` for VM disks and installer ISOs

## Settings it needs to work

- **Log out and back in (or reboot) after installing** — group membership (`libvirt`, `kvm`) only takes effect on next login.
- Drop installer ISOs in `~/.local/share/libvirt/iso/` — Virt-Manager's file picker finds them there automatically.

## Install

```bash
chmod +x install-kvm-qemu.sh
./install-kvm-qemu.sh
```

Then: `virt-manager` to open the GUI and create a VM.

## Removing it afterward

This is meant to be temporary — clean removal instructions are printed at the end of the script:

```bash
virsh undefine <vm-name> --remove-all-storage   # remove the VM + its disk
sudo pacman -Rns qemu-desktop libvirt virt-manager virt-viewer dnsmasq edk2-ovmf
```
