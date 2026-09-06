---
title: NUC 16 Pro — Intel Panther Lake driver-stack
---

# 🇳🇴 Norsk

## NUC 16 Pro — Intel Core Ultra Series 3 (Panther Lake) komplett driver-install

**Maskin:** NUC 16 Pro, Intel Core Ultra Series 3 (Panther Lake, PTL-H484), integrert Xe3-GPU, NPU5.
**Modul:** `20-gpu-drivers/install-intel-panther-lake.sh` i [arch-hypr-dots](https://github.com/archruud/arch-hypr-dots)
**Formål:** Utnytte alt CPU-en kan levere — grafikk, GPU-compute og NPU-basert AI — med rene open source-drivere fra offisielle Arch-repoer.

### Hva er TOPS?

TOPS = Tera Operations Per Second — billioner regneoperasjoner (vanligvis INT8) per sekund. Det måler hvor raskt en AI-akselerator kan kjøre en ferdigtrent modell (inferens), ikke generell CPU-ytelse.

For Panther Lake:
- **NPU5 alene:** opptil 50 TOPS
- **Xe3-GPU:** bidrar også til AI-arbeid
- **Platform-total** (CPU+GPU+NPU): Intel oppgir 100–180 TOPS avhengig av eksakt CPU-modell (386H/388H osv.)

Sjekk din eksakte modell med `dmidecode -t processor | grep Version` for nøyaktig tall.

### Hva installeres og hvorfor

| Pakke | Repo | Hva den gjør |
|---|---|---|
| `intel-npu-driver` | extra | Level Zero userspace-driver for NPU-en (`/dev/accel/accel0`) |
| `intel-npu-compiler` | extra | MLIR/VPUX-kompilator — oversetter OpenVINO-modeller til noe NPU-en forstår |
| `intel-compute-runtime` | extra | Level Zero + OpenCL for **GPU-compute** (separat fra grafikk-driveren) |
| `openvino` | extra | Rammeverk for å faktisk kjøre AI-modeller (Whisper, LLM-er, bildeklassifisering) |
| `openvino-intel-npu-plugin` | extra | Lar OpenVINO snakke med NPU-en spesifikt |
| `libva-utils` | extra | `vainfo` — testverktøy for video-akselerasjon |
| `vulkan-tools` | extra | `vulkaninfo`/`vkcube` — testverktøy for Vulkan |

**Fjernes:** `libva-intel-driver` — gammel i965 VA-API-driver for Gen8–11-GPU-er. Irrelevant for Xe3 og kan forvirre VA-API sitt auto-valg av backend. `intel-media-driver` (iHD) er den som faktisk brukes.

**Allerede på plass fra `01-base`:** `mesa`, `vulkan-intel` (ANV Vulkan-driver), `vulkan-mesa-implicit-layers`, `intel-media-driver`, `intel-gmmlib`, `intel-ucode`, `linux-firmware-intel`.

### Kjernedriver: xe, ikke i915

Panther Lake/Xe3 bruker den nye `xe`-kernel-driveren (ikke den gamle `i915`). Dette er stabilt og på som standard siden kernel 6.17+. Ingen manuell konfigurasjon nødvendig på en oppdatert Arch-installasjon.

Verifiser:
```bash
readlink /sys/bus/pci/devices/0000:00:02.0/driver
# skal ende på .../drivers/xe
```

### ⚠️ Reboot kreves

`intel-npu-driver` sier det rett ut ved install: *"A system reboot is required to start using the Intel NPU driver."* Kernel-modulen (`intel_vpu`) og et eventuelt nytt gruppemedlemskap trer først i kraft etter omstart — testkommandoene under gir feil eller tomt svar før det. Scriptet spør automatisk om å reboote på slutten.

### Verifisering etter install (og reboot)

```bash
ls /dev/accel/                # accel0 skal finnes (NPU)
sudo dmesg | grep -i vpu      # NPU-firmware lastet uten feil (krever sudo — dmesg er restriktert som standard)
vainfo                        # video-akselerasjon (VA-API)
vulkaninfo | grep deviceName  # forventer "Intel(R) Graphics (PTL)"
groups                        # bekreft gruppemedlemskap
```

`/dev/accel/accel0` eies i praksis av gruppen `render` (samme gruppe som GPU-enhetene), ikke world-writable — scriptet legger deg automatisk til denne gruppen om du ikke allerede er medlem.

### SR-IOV og Venus (VM-bruk)

Om du senere skal dele denne GPU-en med en test-VM (Arch+Hyprland i virt-manager), se eget notat om **Venus** (delt virtio-gpu, host og gjest bruker GPU-en samtidig — riktig valg på en stue-PC du selv sitter foran) versus **SR-IOV** (partisjonerer GPU-en i virtuelle funksjoner — riktig for en dedikert server som Proxmox, ikke for denne maskinen).

---

# 🇬🇧 English

## NUC 16 Pro — Intel Core Ultra Series 3 (Panther Lake) complete driver install

**Machine:** NUC 16 Pro, Intel Core Ultra Series 3 (Panther Lake, PTL-H484), integrated Xe3 GPU, NPU5.
**Module:** `20-gpu-drivers/install-intel-panther-lake.sh` in [arch-hypr-dots](https://github.com/archruud/arch-hypr-dots)
**Purpose:** Use everything the CPU can offer — graphics, GPU compute, and NPU-based AI — using only open-source drivers from official Arch repositories.

### What is TOPS?

TOPS = Tera Operations Per Second — trillions of (typically INT8) compute operations per second. It measures how fast an AI accelerator can run an already-trained model (inference), not general CPU performance.

For Panther Lake:
- **NPU5 alone:** up to 50 TOPS
- **Xe3 GPU:** also contributes to AI workloads
- **Platform total** (CPU+GPU+NPU): Intel quotes 100–180 TOPS depending on the exact CPU model (386H/388H etc.)

Check your exact model with `dmidecode -t processor | grep Version` for a precise figure.

### What gets installed and why

| Package | Repo | What it does |
|---|---|---|
| `intel-npu-driver` | extra | Level Zero userspace driver for the NPU (`/dev/accel/accel0`) |
| `intel-npu-compiler` | extra | MLIR/VPUX compiler — translates OpenVINO models into something the NPU understands |
| `intel-compute-runtime` | extra | Level Zero + OpenCL for **GPU compute** (separate from the graphics driver) |
| `openvino` | extra | Framework to actually run AI models (Whisper, LLMs, image classification) |
| `openvino-intel-npu-plugin` | extra | Lets OpenVINO talk to the NPU specifically |
| `libva-utils` | extra | `vainfo` — test tool for video acceleration |
| `vulkan-tools` | extra | `vulkaninfo`/`vkcube` — test tools for Vulkan |

**Removed:** `libva-intel-driver` — the legacy i965 VA-API driver for Gen8–11 GPUs. Irrelevant for Xe3 and can confuse VA-API's backend auto-selection. `intel-media-driver` (iHD) is the one actually in use.

**Already in place from `01-base`:** `mesa`, `vulkan-intel` (ANV Vulkan driver), `vulkan-mesa-implicit-layers`, `intel-media-driver`, `intel-gmmlib`, `intel-ucode`, `linux-firmware-intel`.

### Kernel driver: xe, not i915

Panther Lake/Xe3 uses the newer `xe` kernel driver (not the legacy `i915`). This has been stable and on by default since kernel 6.17+. No manual configuration needed on an up-to-date Arch install.

Verify:
```bash
readlink /sys/bus/pci/devices/0000:00:02.0/driver
# should end in .../drivers/xe
```

### ⚠️ Reboot required

`intel-npu-driver` states it plainly at install time: *"A system reboot is required to start using the Intel NPU driver."* The kernel module (`intel_vpu`) and any new group membership only take effect after a restart — the test commands below will fail or return nothing before that. The script prompts automatically to reboot at the end.

### Verification after install (and reboot)

```bash
ls /dev/accel/                # accel0 should exist (NPU)
sudo dmesg | grep -i vpu      # NPU firmware loaded without errors (needs sudo — dmesg is restricted by default)
vainfo                        # video acceleration (VA-API)
vulkaninfo | grep deviceName  # expect "Intel(R) Graphics (PTL)"
groups                        # confirm group membership
```

`/dev/accel/accel0` is actually owned by the `render` group (same as the GPU devices), not world-writable — the script automatically adds you to this group if you're not already a member.

### SR-IOV and Venus (VM use)

If you later want to share this GPU with a test VM (Arch+Hyprland in virt-manager), see the separate note on **Venus** (shared virtio-gpu, host and guest use the GPU concurrently — the right choice on a living-room PC you sit in front of) versus **SR-IOV** (partitions the GPU into virtual functions — the right choice for a dedicated server like Proxmox, not this machine).
