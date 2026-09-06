# NUC 16 Pro — Intel Panther Lake driver-stack

*[English (primary)](nuc16-panther-lake-drivers.md) — norsk versjon*

## Maskin

NUC 16 Pro, Intel Core Ultra Series 3 (Panther Lake, PTL-H484), integrert Xe3-GPU, NPU5.
**Modul:** `Drivers/Install-nuc16-panther-lake-drivers.sh` i [scripts](https://github.com/archruud/scripts)
**Formål:** Utnytte alt CPU-en kan levere — grafikk, GPU-compute og NPU-basert AI — med rene open source-drivere fra offisielle Arch-repoer.

## Hva er TOPS?

TOPS = Tera Operations Per Second — billioner regneoperasjoner (vanligvis INT8) per sekund. Det måler hvor raskt en AI-akselerator kan kjøre en ferdigtrent modell (inferens), ikke generell CPU-ytelse.

For Panther Lake:
- **NPU5 alene:** opptil 50 TOPS
- **Xe3-GPU:** bidrar også til AI-arbeid
- **Platform-total** (CPU+GPU+NPU): Intel oppgir 100–180 TOPS avhengig av eksakt CPU-modell (386H/388H osv.)

Sjekk din eksakte modell med `dmidecode -t processor | grep Version` for nøyaktig tall.

## Hva installeres og hvorfor

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

## Kjernedriver: xe, ikke i915

Panther Lake/Xe3 bruker den nye `xe`-kernel-driveren (ikke den gamle `i915`). Dette er stabilt og på som standard siden kernel 6.17+. Ingen manuell konfigurasjon nødvendig på en oppdatert Arch-installasjon.

Verifiser:
```bash
readlink /sys/bus/pci/devices/0000:00:02.0/driver
# skal ende på .../drivers/xe
```

## ⚠️ Reboot kreves

`intel-npu-driver` sier det rett ut ved install: *"A system reboot is required to start using the Intel NPU driver."* Kernel-modulen (`intel_vpu`) og et eventuelt nytt gruppemedlemskap trer først i kraft etter omstart — testkommandoene under gir feil eller tomt svar før det. Scriptet spør automatisk om å reboote på slutten.

## Verifisering etter install (og reboot)

```bash
ls /dev/accel/                # accel0 skal finnes (NPU)
sudo dmesg | grep -i vpu      # NPU-firmware lastet uten feil (krever sudo — dmesg er restriktert som standard)
vainfo                        # video-akselerasjon (VA-API)
vulkaninfo | grep deviceName  # forventer "Intel(R) Graphics (PTL)"
groups                        # bekreft gruppemedlemskap
```

`/dev/accel/accel0` eies i praksis av gruppen `render` (samme gruppe som GPU-enhetene), ikke world-writable — scriptet legger deg automatisk til denne gruppen om du ikke allerede er medlem.

## SR-IOV og Venus (VM-bruk)

Om du senere skal dele denne GPU-en med en test-VM (Arch+Hyprland i virt-manager), se eget notat om **Venus** (delt virtio-gpu, host og gjest bruker GPU-en samtidig — riktig valg på en stue-PC du selv sitter foran) versus **SR-IOV** (partisjonerer GPU-en i virtuelle funksjoner — riktig for en dedikert server som Proxmox, ikke for denne maskinen).
