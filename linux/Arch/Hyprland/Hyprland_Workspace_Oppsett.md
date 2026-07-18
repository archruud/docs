# Hyprland Workspace-oppsett

## Mål

Denne guiden beskriver et anbefalt oppsett for Arch Linux + Hyprland
med:

-   Automatisk prioritering av ekstern skjerm
-   Fast workspace-inndeling
-   Waybar som viser riktige workspaces
-   Fungerer både hjemme og på reise

------------------------------------------------------------------------

# Scenario 1 -- Én ekstern skjerm + laptop

## Maskinvare

-   Lenovo 27" 1920×1080@60 (primær)
-   Laptop 1920×1200@60

### Workspace

  Skjerm   Workspaces
  -------- ------------
  Lenovo   1--10
  Laptop   11--20

Når den eksterne skjermen ikke er tilkoblet brukes workspace 1--10
automatisk på laptop.

### Monitor-eksempel

``` conf
monitor=DP-1,1920x1080@60,0x0,1
monitor=eDP-1,1920x1200@60,1920x0,1
```

### Workspace-binding

``` conf
workspace=1,monitor:DP-1
workspace=2,monitor:DP-1
...
workspace=10,monitor:DP-1

workspace=11,monitor:eDP-1
...
workspace=20,monitor:eDP-1
```

### Waybar

På Lenovo vises:

    1 2 3 4 5 6 7 8 9 10

På laptop vises:

    11 12 13 14 15 16 17 18 19 20

------------------------------------------------------------------------

# Scenario 2 -- Tre eksterne skjermer + laptop

## Maskinvare

-   Lenovo 1 (primær)
-   Lenovo 2
-   Lenovo 3
-   Laptop (vanligvis lukket)

### Workspace

  Skjerm     Workspaces
  ---------- ------------
  Lenovo 1   1--10
  Lenovo 2   11--20
  Lenovo 3   21--30
  Laptop     31--40

### Monitor-eksempel

``` conf
monitor=DP-1,1920x1080@60,0x0,1
monitor=HDMI-A-1,1920x1080@60,1920x0,1
monitor=HDMI-A-2,1920x1080@60,3840x0,1
monitor=eDP-1,1920x1200@60,5760x0,1
```

### Workspace-binding

``` conf
workspace=1,monitor:DP-1
...
workspace=10,monitor:DP-1

workspace=11,monitor:HDMI-A-1
...
workspace=20,monitor:HDMI-A-1

workspace=21,monitor:HDMI-A-2
...
workspace=30,monitor:HDMI-A-2

workspace=31,monitor:eDP-1
...
workspace=40,monitor:eDP-1
```

### Waybar

**Lenovo 1**

    1 2 3 4 5 6 7 8 9 10

**Lenovo 2**

    11 12 13 14 15 16 17 18 19 20

**Lenovo 3**

    21 22 23 24 25 26 27 28 29 30

**Laptop**

    31 32 33 34 35 36 37 38 39 40

------------------------------------------------------------------------

# Anbefaling

Den mest fleksible løsningen er:

-   Eksterne skjermer har alltid laveste workspace-numre.
-   Laptop brukes kun når den er alene eller som ekstra skjerm.
-   Waybar viser bare workspaces for den aktuelle skjermen.
-   Et lite Bash-script kan automatisk oppdage hvor mange skjermer som
    er koblet til og velge riktig oppsett uten manuell endring av
    `hyprland.conf`.
