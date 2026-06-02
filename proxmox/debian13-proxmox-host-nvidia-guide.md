# proxmox-host-nvidia-lxc-guide.md

# Proxmox Host NVIDIA GPU Setup for LXC Containers

Denne guiden installerer NVIDIA-driver på Proxmox-host og gjør GPU-er tilgjengelige for LXC-containere.

## 1. Oppdater systemet

```bash
apt update
apt full-upgrade -y
reboot
```

Kontroller kernel:

```bash
uname -r
```

## 2. Installer nødvendige pakker

```bash
apt install -y \
  proxmox-headers-$(uname -r) \
  build-essential \
  dkms
```

## 3. Deaktiver Nouveau

```bash
nano /etc/modprobe.d/blacklist-nouveau.conf
```

Innhold:

```text
blacklist nouveau
options nouveau modeset=0
```

```bash
update-initramfs -u -k all
reboot
```

## 4. Installer NVIDIA-driver

```bash
chmod +x NVIDIA-Linux-x86_64-595.71.05.run
./NVIDIA-Linux-x86_64-595.71.05.run
```

Velg DKMS = Yes.

## 5. Ved kerneloppdatering

```bash
apt install -y proxmox-headers-$(uname -r)
dkms autoinstall
dkms status
```

## 6. Test GPU

```bash
modprobe nvidia
nvidia-smi
```

## 7. LXC GPU Passthrough

Rediger:

```bash
nano /etc/pve/lxc/200.conf
```

Legg til:

```text
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 509:* rwm
lxc.cgroup2.devices.allow: c 234:* rwm

lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia1 dev/nvidia1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-caps dev/nvidia-caps none bind,optional,create=dir
```

## 8. Container

```bash
pct set 200 --features nesting=1,keyctl=1
pct reboot 200
```

## 9. Test inne i container

```bash
ls -la /dev/nvidia*
nvidia-smi
```

Resultat: GPU tilgjengelig i LXC.
