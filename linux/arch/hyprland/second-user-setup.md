# Second User Setup — Arch Linux + Hyprland (Clean Config)

🇳🇴 Norsk versjon: [second-user-setup.no.md](second-user-setup.no.md)

Step-by-step guide for creating a second, fully isolated user (`test`) on an existing Arch Linux + Hyprland install — with sudo rights, access to all system-wide installed applications, but **no** inheritance of the first user's dotfiles/Hyprland config. Intended as a first-time / throwaway test account.

## Goal

- New user `test`, password `123456`
- Full sudo access
- Every system-wide installed app available (pacman/AUR packages, system Flatpaks)
- Completely fresh `~/.config` — none of user 1's Hyprland/Waybar/shell config

## Prerequisites

- You're logged in as user 1 (or root) with sudo working
- `wheel` group is your sudo group (default Arch convention — confirm below)

## Step 1 — Create the user

```bash
sudo useradd -m -G wheel -s /bin/bash test
```

- `-m` creates `/home/test` fresh from `/etc/skel` — a brand new home directory, completely separate from user 1's `/home/<user1>`. This alone guarantees no Hyprland config is inherited, since nothing has been copied.
- `-G wheel` adds `test` to the `wheel` group for sudo.
- `-s /bin/bash` sets the login shell. Swap for `/bin/zsh` or `/usr/bin/fish` if you'd rather test with a different shell (must already be installed).

## Step 2 — Set the password

```bash
sudo passwd test
```

Enter `123456` twice. `passwd` will likely print a `BAD PASSWORD: too simple` warning via `pam_pwquality` — as root this is non-fatal and the password will still be set.

Optional, if you want to be forced to change it on first login instead of using `123456` long-term:

```bash
sudo passwd -e test
```

## Step 3 — Confirm sudo is enabled for `wheel`

```bash
sudo EDITOR=nano visudo
```

Make sure this line is uncommented:

```
%wheel ALL=(ALL:ALL) ALL
```

Verify it works:

```bash
su - test
sudo whoami
# should print: root
exit
```

## Step 4 — Confirm the config is clean

Since `test` has a brand-new home directory, there's nothing to strip out — but it's worth confirming:

```bash
ls -la /home/test/.config 2>/dev/null
```

This should be empty or not exist yet. When Hyprland is launched for `test` with no `~/.config/hypr/hyprland.conf` present, it falls back to Hyprland's own bundled default config — not user 1's config.

**Optional:** if you'd rather `test` start from the stock example config instead of Hyprland's bare internal defaults:

```bash
sudo -u test mkdir -p /home/test/.config/hypr
sudo cp /usr/share/hypr/hyprland.conf /home/test/.config/hypr/hyprland.conf
sudo chown -R test:test /home/test/.config
```

Skip this if you want the purest possible "nothing configured" test.

## Step 5 — Access to installed applications

Nothing extra to do here in most cases: pacman and AUR (yay/paru) packages install into `/usr/bin` and register desktop entries in `/usr/share/applications`, both of which are shared system-wide and visible to every user automatically, including `test`.

Two caveats worth knowing:

| Install method | Visible to `test`? |
|---|---|
| `pacman -S <pkg>` | ✅ Yes — system-wide |
| `yay`/`paru -S <aur-pkg>` | ✅ Yes — installs into `/usr`, system-wide |
| `flatpak install <app>` (default, system remote) | ✅ Yes |
| `flatpak install --user <app>` | ❌ No — only visible to the user who installed it |
| systemd `--user` services set up by user 1 | ❌ No — per-user by design, won't run for `test` |

If user 1 has been installing Flatpaks with `--user`, those specific apps won't show up for `test` — that's expected isolation, not a bug.

## Step 6 — First login into Hyprland

Depends on how Hyprland is launched on this machine:

**Display/login manager (SDDM, greetd, ly, etc.):** `test` appears in the user list automatically once the account exists. Select it, enter `123456`, pick the Hyprland session, log in.

**Manual TTY launch:** switch to a free TTY (e.g. `Ctrl+Alt+F2`), log in as `test`, then run whatever command normally starts the session (`Hyprland`, or `uwsm start hyprland` if UWSM is in use). Since no config exists yet (unless you did the optional Step 4 copy), you'll land in Hyprland's minimal built-in defaults — enough to confirm the session boots, apps launch, and sudo works.

## Cleanup / rollback

To remove the test account and its home directory entirely:

```bash
sudo userdel -r test
```

## ⚠️ Security note

`123456` is a trivially weak password on an account with full sudo rights. Fine for a short, local first-boot test — **not** fine to leave in place, especially on a machine reachable over the network (SSH, VNC, etc.). Either delete the account when you're done testing (`userdel -r` above), or set a real password (`sudo passwd test`) before it sticks around.
