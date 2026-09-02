# 06 — Network & Samba Client

*English (primary) — [norsk versjon tilgjengelig](network-no.md)*

## What it is

Sets up Dolphin as a full network client: SMB/Windows shares, NFS, SSH/SFTP, mDNS discovery (Avahi), mobile connections (MTP/AFC), and helper scripts for scanning and mounting. **Client only** — no server services are started or exposed.

## What it does

- Installs protocol drivers: `smbclient`, `cifs-utils`, `gvfs-smb`, `nfs-utils`, `gvfs-nfs`, `avahi`, `nss-mdns`, `gvfs-mtp`, `gvfs-afc`, `kio-extras`, `kdeconnect`, and more
- (AUR) `kio-gdrive` for Google Drive in Dolphin, if `yay` is present
- Sets up `/etc/samba/smb.conf` as a pure client (no `smbd`/`nmbd` server services)
- Creates mount points: `/mnt/network`, `/mnt/smb`, `/mnt/nfs`, `/mnt/ftp`
- Adds network shortcuts to Dolphin's sidebar (`smb:/`, `ftp:/`, `remote:/`)
- Installs three helper commands in `~/.local/bin/`:
  - `scan-network` — scans all defined VLANs (10, 20, 30, 40, 50, 75) + mDNS + NetBIOS
  - `mount-smb //server/share mountpoint [user] [password]` — mounts an SMB share, tries SMB 3.0 → 2.1 → 2.0
  - `test-protocols` — checks that everything is installed and running
- Stops and disables any server services (`smbd`, `nmbd`, `winbind`, `vsftpd`) to guarantee pure client mode

## Settings it needs to work

- **The VLAN numbers in `scan-network`** (10/20/30/40/50/75) are hardcoded to your UniFi setup — adjust in the script if the VLAN plan changes
- **Keybinds must be added to `hyprland.lua` manually** — the script checks whether they exist and prints the four lines if not (see below). It never writes to `hyprland.conf` anymore (unused since the Lua migration)

```lua
hl.bind(mainMod .. " + N",         hl.dsp.exec_cmd("dolphin smb:/"))
hl.bind(mainMod .. " + SHIFT + N", hl.dsp.exec_cmd("dolphin remote:/"))
hl.bind(mainMod .. " + CTRL + N",  hl.dsp.exec_cmd("dolphin mtp:/"))
hl.bind(mainMod .. " + ALT + N",   hl.dsp.exec_cmd("~/.local/bin/scan-network"))
```

## Changes from the original

- The two identical files (`install-network_script.sh` and `network_script.sh`) were merged into one: `install-network.sh`
- The keybind section no longer writes blindly to `hyprland.conf` — it checks `hyprland.lua` and shows you the four lines to add yourself
- The log filename is now consistently `install-06-network-<date>.log`

## Install

```bash
chmod +x install-network.sh
./install-network.sh
```

The full log is saved automatically to `~/install-06-network-<date>.log`. At the end, the script asks whether you want to run `scan-network` and open the SMB browser in Dolphin right away.
