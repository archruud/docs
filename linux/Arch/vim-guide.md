# Vim for Nybegynnere - Komplett Guide

**For:** Archruud | **Oppdatert:** 2026  
**Merk:** Spesialtips for venstrehente inkludert!

---

## Hva er Vim?

Vim er en tekstredaktør som kjører i terminalen. Den virker komplisert, men når du lærer de grunnleggende kommandoene er den ekstremt rask.

**Viktigste konsept:** Vim har MODUSER. Du veksler mellom å navigere og å skrive.

---

## De 3 viktigste modusene

| Modus | Hva det er | Hvordan komme inn |
|-------|-----------|-------------------|
| **Normal** | Navigasjon og kommandoer | `Esc` (alltid!) |
| **Insert** | Skrive tekst | `i` |
| **Command** | Lagre, avslutte osv | `:` |

**Huskeregel:** Trykk `Esc` når du er i tvil. Du er alltid trygg i Normal-modus.

---

## Starte og avslutte (det viktigste!)

```
vim fil.txt         # Åpne fil
vim ny-fil.txt      # Lag ny fil
```

### Avslutte Vim (alle metoder):

```
:q          # Avslutt (hvis ingen endringer)
:q!         # Tvangsavslutt UTEN å lagre
:w          # Lagre
:wq         # Lagre og avslutt
:x          # Lagre og avslutt (kortere)
ZZ          # Lagre og avslutt (uten kolon!)
ZQ          # Avslutt uten å lagre (uten kolon!)
```

---

## Din første økt - steg for steg

```bash
# 1. Åpne en testfil
vim test.txt

# Du er nå i Normal-modus (ingenting skjer når du skriver)

# 2. Trykk 'i' for å begynne å skrive
i

# 3. Skriv noe tekst
Hei verden!
Dette er min første Vim-fil.

# 4. Trykk Esc for å gå tilbake til Normal-modus
[Esc]

# 5. Lagre og avslutt
:wq
[Enter]
```

**Gratulerer! Du har brukt Vim!** 🎉

---

## Navigasjon i Normal-modus

### Piltaster vs Vim-taster

Piltastene fungerer i Vim, men de "ekte" Vim-tastene er raskere:

```
h   ←  (venstre)
j   ↓  (ned)
k   ↑  (opp)
l   →  (høyre)
```

**Tips for venstrehente:** Du trenger ikke bruke hjkl — piltastene fungerer helt fint!

### Raskere navigasjon

```
w           # Hopp til neste ORD
b           # Hopp til forrige ORD
e           # Hopp til slutt av ORD
0           # Gå til begynnelse av linje
$           # Gå til slutt av linje
gg          # Gå til TOPP av fil
G           # Gå til BUNN av fil
:50         # Gå til linje 50
Ctrl+d      # Rull ned halvt skjermbilde
Ctrl+u      # Rull opp halvt skjermbilde
```

---

## Redigere tekst

### Gå inn i Insert-modus

```
i       # Insert FORAN markøren (mest brukt)
a       # Insert ETTER markøren
I       # Insert på BEGYNNELSE av linje
A       # Insert på SLUTT av linje
o       # Ny linje UNDER og insert
O       # Ny linje OVER og insert
```

### Slette tekst (i Normal-modus)

```
x       # Slett ett tegn
dd      # Slett hele linjen
d2d     # Slett 2 linjer
dw      # Slett ett ord
d$      # Slett fra markør til slutt av linje
D       # Samme som d$
```

### Kopiere og lime inn

```
yy      # Kopier (yank) hele linjen
y2y     # Kopier 2 linjer
yw      # Kopier ett ord
p       # Lim inn ETTER markøren
P       # Lim inn FØR markøren
```

### Angre og gjøre om

```
u           # Angre (Undo) - bruk mange ganger!
Ctrl+r      # Gjør om (Redo)
.           # Gjenta siste handling
```

---

## Søke og erstatte

```
/søkeord        # Søk fremover
?søkeord        # Søk bakover
n               # Neste treff
N               # Forrige treff
```

### Erstatte tekst

```
:s/gammelt/nytt/        # Erstatt første på linjen
:s/gammelt/nytt/g       # Erstatt alle på linjen
:%s/gammelt/nytt/g      # Erstatt ALLE i filen
:%s/gammelt/nytt/gc     # Erstatt alle, spør for hver
```

---

## Praktiske tips

```
:set number         # Vis linjenummer
:set nonumber       # Skjul linjenummer
:set paste          # Lim inn uten automatisk innrykk (nyttig!)
:set nopaste        # Av igjen

gg=G                # Auto-formater hele filen
==                  # Auto-formater gjeldende linje
```

### Lagre en skrivebeskyttet fil med sudo

```
:w !sudo tee %
```

---

## Tips for venstrehente

Vim er designet for høyrehente (hjkl er på høyre side). Som venstrehent kan du:

1. **Bruk piltastene** — det er helt OK! Det er bare litt tregere.
2. **Remapp tastene** i `~/.vimrc`:

```vim
" ~/.vimrc - venstrehent oppsett
" Bruk WASD for navigasjon
noremap w k
noremap a h
noremap s j
noremap d l
```

3. **Lær kommandoene gradvis** — du trenger ikke lære alt på én gang.

---

## Minimal ~/.vimrc (anbefalt)

Lag filen `~/.vimrc` med dette innholdet:

```vim
" Grunnleggende innstillinger
set number          " Linjenummer
set relativenumber  " Relative linjenummer
set hlsearch        " Highlight søkeresultater
set incsearch       " Søk mens du skriver
set ignorecase      " Søk uten case-sensitivitet
set smartcase       " Men case-sensitiv hvis du bruker STORE bokstaver
set tabstop=4       " Tab = 4 mellomrom
set shiftwidth=4    " Innrykk = 4 mellomrom
set expandtab       " Bruk mellomrom i stedet for tab
set autoindent      " Auto-innrykk
set wrap            " Brekk lange linjer
set mouse=a         " Aktiver mus
syntax on           " Syntaksfarger
```

---

## Rask referanse

```
Esc         Normal-modus (alltid trygt)
i           Skriv tekst
:wq         Lagre og avslutt
:q!         Avslutt uten å lagre
u           Angre
dd          Slett linje
yy/p        Kopier/Lim inn linje
/søk        Søk
n           Neste treff
gg/G        Topp/Bunn av fil
```

---

*Sist oppdatert: 2026 | archruud.org*
