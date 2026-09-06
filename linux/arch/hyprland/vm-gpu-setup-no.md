# VM GPU-akselerasjon — Oppsettsguide (virt-manager, archmini)

*[English (primary)](vm-gpu-setup.md) — norsk versjon*

*Følgesvenn til `kvm-qemu-setup-no.md` — det dokumentet dekker installasjon av selve KVM/QEMU; dette dekker å få GPU-akselerasjon til å fungere inni en spesifikk VM.*

## Hvorfor ikke SR-IOV eller full PCI-passthrough

- **SR-IOV** finnes ikke for denne maskinvaren. Intels gamle GVT-g-ordning for iGPU-er ble fjernet fra kernel for flere år siden, og Panther Lake sin Xe3 har aldri støttet det. Ikke "vanskelig" — rett og slett ikke tilgjengelig.
- **Full VFIO PCI-passthrough** ville krevd en dedikert ekstra GPU til verten, siden hele GPU-en overlates eksklusivt til VM-en. archmini har bare den ene integrerte Xe3-en — å passtrøse den ville latt verten stå uten skjermbilde.
- **Det du faktisk vil ha** (én VM om gangen, du sitter foran verten): **Venus** — en delt virtio-gpu som både vert og gjest bruker samtidig over Vulkan. Ingen IOMMU/VFIO-oppsett, verten mister aldri skjermbildet.

## Vert-siden — virt-manager-innstillinger

I VM-ens maskinvaredetaljer (`Add Hardware` / eksisterende enhetsliste):

| Enhet | Innstilling |
|---|---|
| Video | Model: **Virtio**, ☑ **3D acceleration** |
| Display | Type: **Spice server**, Listen type: **None**, ☑ **OpenGL**, GPU: velg din faktiske enhet (f.eks. `0000:00:02:0 Intel...`) |

Ingenting annet trenger installeres på selve NUC-verten — dette er VM-konfigurasjon, ikke vert-pakker.

## Gjest-siden — hvilken driver som skal velges i `archinstall`

Ingen av de to "åpenbare" valgene er riktig for en KVM/QEMU-gjest:

- ❌ **"Intel (Open-source)"** — feil. Gjesten ser aldri ekte Intel-maskinvare, bare en virtuell `virtio-gpu`-enhet. Denne profilen drar inn `intel-media-driver`/`libva-intel-driver`, som ikke gjør noe nyttig her (ufarlig, bare bortkastet).
- ❌ **"VMware / VirtualBox (Open-source)"** — helt feil hypervisor. Drar inn `open-vm-tools`/VirtualBox-gjestetillegg, som er for VMware/VirtualBox, ikke QEMU/KVM.
- ✅ **"All open-source"** — riktig. Generisk `mesa` + `vulkan-icd-loader`, ingen produsent-innlåsing. Selve `virtio-gpu`-kernel-driveren ligger allerede innebygd i Linux-kjernen — ingen egen pakke nødvendig for den delen.

## Gjest-siden — kjør etter første boot

`install-vm-guest-gpu.sh` (kjøres **inni gjesten**, aldri på verten):

```bash
chmod +x install-vm-guest-gpu.sh
./install-vm-guest-gpu.sh
```

Hva den gjør:
- Bekrefter at den faktisk kjører i en VM (nekter å kjøre på bare metall, som sikkerhetssjekk)
- Installerer `mesa`, `vulkan-icd-loader`, `vulkan-mesa-implicit-layers`, `qemu-guest-agent`
- Aktiverer `qemu-guest-agent` (trengs for ren avslutning/status-rapportering fra virt-manager — du har allerede `Channel (qemu-ga)`-enheten konfigurert på vert-siden, dette er den manglende gjeste-halvparten)
- Kjører `vulkaninfo` og sjekker at resultatet ikke er `llvmpipe` (som ville betydd ren software-rendering — ingen GPU-akselerasjon i det hele tatt)

## Feilsøking

Hvis `deviceName` viser `llvmpipe` i stedet for en virtio/Venus-enhet, er den vanligste årsaken at **OpenGL-boksen under Display → Spice** ikke er krysset av på vert-siden, eller at feil GPU er valgt i samme nedtrekksmeny. Dobbeltsjekk vert-tabellen over før du antar at gjeste-pakkene er problemet.
