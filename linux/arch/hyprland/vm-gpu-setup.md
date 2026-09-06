# VM GPU Acceleration — Setup Guide (virt-manager, archmini)

*English (primary) — [norsk versjon tilgjengelig](vm-gpu-setup-no.md)*

*Companion to `kvm-qemu-setup.md` — that doc covers installing KVM/QEMU itself; this one covers getting GPU acceleration working inside a specific VM.*

## Why not SR-IOV or full PCI passthrough

- **SR-IOV** doesn't exist for this hardware. Intel's old GVT-g scheme for iGPUs was removed from the kernel years ago, and Panther Lake's Xe3 never supported it. Not "hard" — not available at all.
- **Full VFIO PCI passthrough** would require a dedicated second GPU for the host, since the whole GPU gets handed to the VM exclusively. archmini only has the one integrated Xe3 — passing it through would leave the host with no display.
- **What you actually want** (single VM at a time, sitting in front of the host): **Venus** — a shared virtio-gpu that both host and guest use concurrently over Vulkan. No IOMMU/VFIO setup, no host display loss.

## Host side — virt-manager settings

In the VM's hardware details (`Add Hardware` / existing device list):

| Device | Setting |
|---|---|
| Video | Model: **Virtio**, ☑ **3D acceleration** |
| Display | Type: **Spice server**, Listen type: **None**, ☑ **OpenGL**, GPU: select your actual device (e.g. `0000:00:02:0 Intel...`) |

Nothing else needs installing on the NUC host itself — these are VM configuration settings, not host packages.

## Guest side — which driver to pick in `archinstall`

Neither of the two "obvious" choices is correct for a KVM/QEMU guest:

- ❌ **"Intel (Open-source)"** — wrong. The guest never sees real Intel hardware, only a virtual `virtio-gpu` device. This profile pulls in `intel-media-driver`/`libva-intel-driver`, which do nothing useful here (harmless, just wasted).
- ❌ **"VMware / VirtualBox (Open-source)"** — wrong hypervisor entirely. Pulls in `open-vm-tools`/VirtualBox guest additions, which are for VMware/VirtualBox, not QEMU/KVM.
- ✅ **"All open-source"** — correct. Generic `mesa` + `vulkan-icd-loader`, no vendor lock-in. The `virtio-gpu` kernel driver itself is already built into the Linux kernel — no separate package needed for that part.

## Guest side — run after first boot

`install-vm-guest-gpu.sh` (run **inside the guest**, never on the host):

```bash
chmod +x install-vm-guest-gpu.sh
./install-vm-guest-gpu.sh
```

What it does:
- Confirms it's actually running in a VM (refuses to run on bare metal, as a safety check)
- Installs `mesa`, `vulkan-icd-loader`, `vulkan-mesa-implicit-layers`, `qemu-guest-agent`
- Enables `qemu-guest-agent` (needed for clean shutdown/status reporting from virt-manager — you already have the `Channel (qemu-ga)` device configured on the host side, this is the missing guest-side half)
- Runs `vulkaninfo` and checks the result isn't `llvmpipe` (which would mean pure software rendering — no GPU acceleration at all)

## Troubleshooting

If `deviceName` shows `llvmpipe` instead of a virtio/Venus device, the most common cause is the **OpenGL checkbox in Display → Spice** not being enabled on the host side, or the wrong GPU selected in that same dropdown. Double-check the host-side table above before assuming the guest packages are the problem.
