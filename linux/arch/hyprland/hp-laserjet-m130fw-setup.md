# HP LaserJet MFP M130fw — Arch Linux CLI Setup

> Network printer install via CUPS, no GUI wizard required.
> Verified working end-to-end on Arch Linux / Hyprland, kitty + bash,
> September 2026.

## TL;DR — confirmed working config

- **Device:** HP LaserJet MFP M130fw, hostname `NPIE03318`, static IP
  `192.168.30.25` (set manually in the printer's own EWS, not DHCP)
- **Working method:** CUPS + HPLIP + `hpcups` PPD via `foomatic-db`.
  **Driverless (IPP Everywhere / `-m everywhere`) does NOT work on this
  unit's firmware** — it accepts the job over IPP but the printer's own
  raster processor throws `URP ERROR / NotImplemented / urp_urf_processor.c`
  and produces a blank/error page. Don't waste time on §3a below unless
  HP ships a firmware update — go straight to hpcups.
- **Queue name used:** `HP_LaserJet_M130fw`
- **PPD used:** `drv:///hp/hpcups.drv/hp-laserjet_mfp_m129-m134.ppd`
- **Device URI used:** `hp:/net/HP_LaserJet_MFP_M129-M134?ip=192.168.30.25`
- **Plugin:** not required for this unit.
- Plain text/PDF jobs print fine. The bundled CUPS
  `/usr/share/cups/data/testprint` file is unreliable as a smoke test on
  this setup (see Troubleshooting) — use a normal document instead.

Working install command, copy-paste ready:

```bash
sudo pacman -S --needed cups hplip sane avahi nss-mdns foomatic-db
sudo systemctl enable --now cups.service
sudo systemctl enable --now avahi-daemon.service

# Required for .local mDNS resolution — add mdns_minimal before resolve
# on the hosts: line in /etc/nsswitch.conf if not already present:
#   hosts: mymachines mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] files myhostname dns
sudo sed -i.bak 's/^hosts:\(.*\)resolve /hosts:\1mdns_minimal [NOTFOUND=return] resolve /' /etc/nsswitch.conf

sudo lpadmin -p HP_LaserJet_M130fw \
  -E \
  -v "hp:/net/HP_LaserJet_MFP_M129-M134?ip=192.168.30.25" \
  -m "drv:///hp/hpcups.drv/hp-laserjet_mfp_m129-m134.ppd" \
  -L "Home Office" \
  -D "HP LaserJet MFP M130fw"

sudo cupsenable HP_LaserJet_M130fw
sudo cupsaccept HP_LaserJet_M130fw
sudo lpadmin -d HP_LaserJet_M130fw   # set as default

echo "test" > /tmp/printtest.txt
lp -d HP_LaserJet_M130fw /tmp/printtest.txt
```

## Prerequisites

- Printer already on the network and reachable (`ping 192.168.30.25`)
- The printer's IP known — check `System > TCP/IP` in the embedded web
  server (`http://192.168.30.25`), or your UniFi DHCP leases / static
  reservations.
- `sudo` access on the Arch box.

## 1. Packages

```bash
sudo pacman -S --needed cups hplip sane avahi nss-mdns foomatic-db
```

| Package       | Purpose                                         |
|---------------|--------------------------------------------------|
| `cups`        | Print spooler / server                          |
| `hplip`       | HP's driver suite (`hp-setup`, `hp-makeuri`, `hp-plugin`) |
| `sane`        | Scanning support (M130fw is a multi-function unit) |
| `avahi`       | mDNS/Bonjour discovery on the LAN               |
| `nss-mdns`    | Resolves `.local` hostnames via avahi           |
| `foomatic-db` | **Not** pulled in automatically by `hplip` — it's the actual driver database. Without it, `lpinfo -m` only returns unusable `driverless:` placeholder entries for this model, which fails with `Missing PPD-Adobe-4.x header`. |

## 2. Enable services

```bash
sudo systemctl enable --now cups.service
sudo systemctl enable --now avahi-daemon.service
```

Fix `.local` resolution — needed even for the hpcups path, since
`hp-makeuri`/discovery still touch mDNS:

```bash
grep '^hosts:' /etc/nsswitch.conf
sudo sed -i.bak 's/^hosts:\(.*\)resolve /hosts:\1mdns_minimal [NOTFOUND=return] resolve /' /etc/nsswitch.conf
```

Optional but recommended: add yourself to the `sys` group so CUPS admin
commands don't need `sudo` every time:

```bash
sudo usermod -aG sys $USER
# log out / back in for it to take effect
```

## 3a. Driverless (IPP Everywhere) — DOES NOT WORK on this unit

The printer advertises IPP Everywhere over mDNS and `lpadmin -m
everywhere` *adds* the queue without error, but every real print job
fails on-device with:

```
URP ERROR
Subsystem: PARSER
Error: NotImplemented
File Name: urp_urf_processor.c
Line Number: 753
```

This is a firmware-side URF/raster-processor bug, not a CUPS
misconfiguration — nothing to fix on the Arch side. Skip straight to
§3b/§4.

## 3b. HPLIP + foomatic PPD (the method that works)

```bash
hp-makeuri 192.168.30.25
```

Output:

```
CUPS URI: hp:/net/HP_LaserJet_MFP_M129-M134?ip=192.168.30.25
SANE URI: hpaio:/net/HP_LaserJet_MFP_M129-M134?ip=192.168.30.25
```

Find the real (non-driverless) PPD:

```bash
lpinfo -m | grep -viE '^driverless' | grep -iE 'm129.?m134|m127.?m128' | grep -i hpcups
```

Expect two candidates; use the `drv:///hp/hpcups.drv/...` one (the
`lsb/usr/HP/...ppd.gz` one also works but is the legacy path):

```
drv:///hp/hpcups.drv/hp-laserjet_mfp_m129-m134.ppd HP LaserJet MFP m129-m134, hpcups 3.26.4
lsb/usr/HP/hp-laserjet_mfp_m129-m134.ppd.gz HP LaserJet MFP m129-m134, hpcups 3.26.4
```

## 4. Add the printer to CUPS

```bash
sudo lpadmin -p HP_LaserJet_M130fw \
  -E \
  -v "hp:/net/HP_LaserJet_MFP_M129-M134?ip=192.168.30.25" \
  -m "drv:///hp/hpcups.drv/hp-laserjet_mfp_m129-m134.ppd" \
  -L "Home Office" \
  -D "HP LaserJet MFP M130fw"

sudo cupsenable HP_LaserJet_M130fw
sudo cupsaccept HP_LaserJet_M130fw
```

Note: `lpadmin` prints `Printer drivers are deprecated and will stop
working in a future version of CUPS.` — this is just a forward-looking
warning about PPD-based drivers in general, not an error. Safe to
ignore for now; if this stops working after a major CUPS upgrade,
that's the reason.

## 5. Plugin (only if required)

```bash
lpinfo -m | grep -i "hp-laserjet_mfp_m129-m134" | grep -i proprietary
```

If it reports `requires proprietary plugin`:

```bash
sudo hp-plugin -i
```

Not required on the unit this guide was verified against.

## 6. Test

**Don't use `/usr/share/cups/data/testprint`** — see Troubleshooting.
Use a plain file instead:

```bash
echo "HP LaserJet M130fw print test - $(date)" > /tmp/printtest.txt
lp -d HP_LaserJet_M130fw /tmp/printtest.txt
```

Set as system default:

```bash
sudo lpadmin -d HP_LaserJet_M130fw
```

Web management UI: `http://localhost:631/printers/HP_LaserJet_M130fw`

## 7. Scanning (SANE) — not yet verified

```bash
scanimage -L                 # should list the hpaio device
sudo pacman -S --needed simple-scan   # optional GUI scan front-end
```

## Automated install script

`install-hp-m130fw.sh` tries driverless first, then falls back to the
hpcups path automatically if no driverless URI is found at *install*
time. It currently can't detect the firmware-level URF failure (that
only shows up when an actual job is sent, not when the queue is
added) — if driverless silently "succeeds" but prints come out as
`URP ERROR` pages, re-run pointing it at the hpcups path manually
using the TL;DR block above.

```bash
chmod +x install-hp-m130fw.sh
./install-hp-m130fw.sh 192.168.30.25 HP_LaserJet_M130fw
```

## Troubleshooting

- **Prints come out as a page reading `URP ERROR / Subsystem: PARSER /
  Error: NotImplemented / urp_urf_processor.c`:** driverless/IPP
  Everywhere firmware bug on this unit — see §3a. Fix: use the hpcups
  PPD (§3b/§4) instead of `-m everywhere`.
- **`Missing PPD-Adobe-4.x header on line 0` from `lpadmin`:** you
  passed a `driverless:ipps://...` string from `lpinfo -m` as `-m`
  directly — not a valid standalone model argument. Install
  `foomatic-db` and use a real `hpcups` PPD instead.
- **`lpadmin: Unable to connect to NPIE03318.local:631: Name or
  service not known`:** `nss-mdns` isn't wired into NSS yet. Fix the
  `hosts:` line in `/etc/nsswitch.conf` (see §2), then retry.
- **Job accepted (`request id is ...`) but nothing happens, no log
  entries for that job:** check `lpstat -l -o <queue>` and `sudo tail
  -40 /var/log/cups/error_log` — cross-reference the job number
  (`[Job N]`) in the log. If the built-in `testprint` file is
  involved, its MIME type resolves to `application/vnd.cups-pdf-banner`
  (CUPS treats it as a banner template, not plain content) and can
  fail through `pdftopdf`/`bannertopdf` even when the printer/driver
  themselves are fine — this happened during initial setup and was a
  red herring. Test with a plain text or PDF file instead.
- **`hp-makeuri` finds nothing:** confirm the printer is reachable
  (`ping`), and that it's genuinely on the same subnet/VLAN — check
  UniFi VLAN/firewall rules aren't blocking mDNS (UDP 5353) or the
  printer's port 9100/631.
- **`hp-setup` crashes on `hpfax://`:** known HPLIP issue with fax
  support enabled and no plugin installed — disable fax on the printer
  itself if you don't use it, per the Arch Wiki.
- **No PPD match in `lpinfo -m`:** confirm `foomatic-db` is installed
  (`pacman -Q foomatic-db`), then re-check with `hp-check -t` for
  other missing dependencies.

## References

- Arch Wiki: [CUPS/Printer-specific problems](https://wiki.archlinux.org/title/CUPS/Printer-specific_problems)
- HPLIP supported devices: https://developers.hp.com/hp-linux-imaging-and-printing/supported_devices/index
