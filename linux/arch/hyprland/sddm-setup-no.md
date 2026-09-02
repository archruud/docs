# 03 — SDDM (Innloggingsskjerm)

*[English (primary)](sddm-setup.md) — norsk versjon*

## Hva det er

Installerer et tilpasset SDDM-tema ("archruud") — en kopi av Elarun-temaet (innloggingsskjema på venstre side, som matcher det du ønsker) med din egen bakgrunn satt inn.

## Hvorfor bakgrunnen forsvant

Dette skyldtes **ikke** en Hyprland- eller Lua-relatert endring — SDDM kjører som sin egen systembruker (`sddm`) og starter *før* Hyprland-sesjonen din i det hele tatt eksisterer. Den vet ingenting om `hyprland.lua`.

Den egentlige årsaken: SDDM-greeteren klarer ikke å lese inn i hjemmemappa di. `/home/archruud` har normalt `700`-rettigheter (kun du kommer inn), og `sddm`-brukeren er ikke deg. En bakgrunnssti som peker inn i `$HOME` — direkte eller indirekte — feiler stille så snart noe (en oppdatering, en tilbakestilt rettighet) eksponerer dette. Dette er en velkjent, godt dokumentert SDDM-oppførsel, ikke en regresjon spesifikk for ditt oppsett.

Fiksen: kopier bakgrunnen inn i selve temamappen under `/usr/share/sddm/themes/archruud/images/background.png` — et sted alle brukere (inkludert `sddm`) kan lese.

## Config-filplasseringer (bekreftet mot `sddm.conf(5)`)

| Sti | Formål |
|---|---|
| `/usr/lib/sddm/sddm.conf.d/` | Systemstandarder — **rediger aldri** |
| `/etc/sddm.conf.d/` | Lokale endringer — **rediger her** |
| `/etc/sddm.conf` | Eldre enkeltfil-alternativ, fungerer fortsatt men mindre ryddig |

Dette scriptet skriver til `/etc/sddm.conf.d/theme.conf`.

## Hva det gjør

- Installerer `sddm` + Qt6-avhengigheter hvis de mangler
- Kopierer `archruud`-temaet (Elarun-basert) til `/usr/share/sddm/themes/archruud`
- Kopierer din nåværende bakgrunn (`~/.config/hypr/wallpapers/ARCHRUUD_2560x1440.png`) inn i temaets `images/background.png`
- Setter globale lese-rettigheter på temamappa (den faktiske fiksen på rettighetsfellen over)
- Skriver `/etc/sddm.conf.d/theme.conf` med `Current=archruud`
- Aktiverer `sddm.service`

## Innstillinger den trenger for å fungere

- Kjør `02-awww` først så bakgrunnsfila finnes.
- Når du bytter skrivebordsbakgrunn og vil at SDDM skal matche, kopier den inn på nytt:
  ```bash
  sudo cp ~/.config/hypr/wallpapers/ARCHRUUD_2560x1440.png /usr/share/sddm/themes/archruud/images/background.png
  ```

## Installasjon

```bash
chmod +x install-sddm-theme.sh
./install-sddm-theme.sh
```

## Filer i denne mappen

```
03-sddm/
├── install-sddm-theme.sh
└── theme/archruud/    (Elarun, omdøpt + pekt til din bakgrunn)
```
