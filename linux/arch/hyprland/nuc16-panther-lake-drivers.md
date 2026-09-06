# NUC 16 Pro — Intel Panther Lake Driver Stack

*English (primary) — [norsk versjon tilgjengelig](nuc16-panther-lake-drivers-no.md)*

## Machine

NUC 16 Pro, Intel Core Ultra Series 3 (Panther Lake, PTL-H484), integrated Xe3 GPU, NPU5.
**Module:** `Drivers/Install-nuc16-panther-lake-drivers.sh` in [scripts](https://github.com/archruud/scripts)
**Purpose:** Use everything the CPU can offer — graphics, GPU compute, and NPU-based AI — using only open-source drivers from official Arch repositories.

## What is TOPS?

TOPS = Tera Operations Per Second — trillions of (typically INT8) compute operations per second. It measures how fast an AI accelerator can run an already-trained model (inference), not general CPU performance.

For Panther Lake:
- **NPU5 alone:** up to 50 TOPS
- **Xe3 GPU:** also contributes to AI workloads
- **Platform total** (CPU+GPU+NPU): Intel quotes 100–180 TOPS depending on the exact CPU model (386H/388H etc.)

Check your exact model with `dmidecode -t processor | grep Version` for a precise figure.

## What gets installed and why

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

## Kernel driver: xe, not i915

Panther Lake/Xe3 uses the newer `xe` kernel driver (not the legacy `i915`). This has been stable and on by default since kernel 6.17+. No manual configuration needed on an up-to-date Arch install.

Verify:
```bash
readlink /sys/bus/pci/devices/0000:00:02.0/driver
# should end in .../drivers/xe
```

## ⚠️ Reboot required

`intel-npu-driver` states it plainly at install time: *"A system reboot is required to start using the Intel NPU driver."* The kernel module (`intel_vpu`) and any new group membership only take effect after a restart — the test commands below will fail or return nothing before that. The script prompts automatically to reboot at the end.

## Verification after install (and reboot)

```bash
ls /dev/accel/                # accel0 should exist (NPU)
sudo dmesg | grep -i vpu      # NPU firmware loaded without errors (needs sudo — dmesg is restricted by default)
vainfo                        # video acceleration (VA-API)
vulkaninfo | grep deviceName  # expect "Intel(R) Graphics (PTL)"
groups                        # confirm group membership
```

`/dev/accel/accel0` is actually owned by the `render` group (same as the GPU devices), not world-writable — the script automatically adds you to this group if you're not already a member.

## SR-IOV and Venus (VM use)

If you later want to share this GPU with a test VM (Arch+Hyprland in virt-manager), see the separate note on **Venus** (shared virtio-gpu, host and guest use the GPU concurrently — the right choice on a living-room PC you sit in front of) versus **SR-IOV** (partitions the GPU into virtual functions — the right choice for a dedicated server like Proxmox, not this machine).
