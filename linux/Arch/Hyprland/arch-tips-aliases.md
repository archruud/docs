# Arch Linux Tips og Aliaser

**For:** Archruud | **Oppdatert:** 2026  
**Shell:** bash/zsh

---

## Nyttige aliaser for ~/.bashrc eller ~/.zshrc

```bash
# === FILBEHANDLING ===
alias ls='ls --color=auto'
alias ll='ls -lah'
alias la='ls -la'
alias l='ls -lh'
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'

# === PAKKEHÅNDTERING ===
alias update='sudo pacman -Syu'
alias install='sudo pacman -S'
alias remove='sudo pacman -Rs'
alias search='pacman -Ss'
alias aur='yay -S'
alias aurlog='yay -Syu'
alias cleanup='sudo pacman -Rns $(pacman -Qtdq)'  # Fjern foreldreløse pakker

# === SYSTEMD ===
alias start='sudo systemctl start'
alias stop='sudo systemctl stop'
alias restart='sudo systemctl restart'
alias status='systemctl status'
alias enable='sudo systemctl enable --now'
alias disable='sudo systemctl disable'
alias logs='journalctl -f'

# === HYPRLAND ===
alias hyprconf='vim ~/.config/hypr/hyprland.conf'
alias waybarconf='vim ~/.config/waybar/config.jsonc'
alias reloadbar='pkill waybar && waybar &'
alias reloaddunst='pkill dunst && dunst &'

# === NETTVERK ===
alias ip4='ip -4 addr'
alias ip6='ip -6 addr'
alias myip='curl ifconfig.me'
alias pingtest='ping -c 4 8.8.8.8'
alias portscan='ss -tuln'

# === DIVERSE ===
alias c='clear'
alias h='history'
alias df='df -h'
alias du='du -sh'
alias free='free -h'
alias top='btop'  # Forutsetter btop installert
alias cat='bat'   # Forutsetter bat installert
alias grep='grep --color=auto'

# === GIT ===
alias gs='git status'
alias ga='git add .'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline'
alias gpl='git pull'

# === SSH SNARVEIER ===
# Legg til dine egne servere her:
# alias docs='ssh archruud@192.168.30.52'
# alias prox='ssh root@192.168.50.10'
```

---

## Legg til aliaser

```bash
# Åpne .bashrc (bash) eller .zshrc (zsh)
vim ~/.bashrc
# eller
vim ~/.zshrc

# Lim inn aliasene du ønsker, lagre

# Last inn endringene uten å restarte terminalen:
source ~/.bashrc
# eller
source ~/.zshrc
```

---

## Nyttige bashrc/zshrc innstillinger

```bash
# === HISTORIKK ===
HISTSIZE=10000
HISTFILESIZE=20000
HISTCONTROL=ignoredups:erasedups  # Ikke dupliser i historikk

# === AUTO-KOMPLETTERING ===
# For bash:
[[ $PS1 && -f /usr/share/bash-completion/bash_completion ]] && \
    . /usr/share/bash-completion/bash_completion

# === PROMPT (minimal stil) ===
# Bash: vis kun mappenavn
PS1='\W \$ '

# Zsh: enkel prompt
PROMPT='%1~ $ '
```

---

## Pacman keyring fix (kjøres ved signaturfeil)

```bash
sudo pacman -Sy archlinux-keyring
sudo pacman-key --init
sudo pacman-key --populate archlinux
```

Lag dette som et alias:
```bash
alias fixkeys='sudo pacman -Sy archlinux-keyring && sudo pacman-key --init && sudo pacman-key --populate archlinux'
```

---

## exa - Bedre ls (installer: yay -S exa)

```bash
alias ls='exa --color=always'
alias ll='exa -lah --color=always --icons'
alias lt='exa -T --color=always'         # Tree-visning
alias la='exa -la --color=always'
```

---

## bat - Bedre cat (installer: pacman -S bat)

```bash
alias cat='bat'
alias catp='bat --plain'     # Uten linjenummer/highlighting
```

---

## WireGuard aliaser (for VPN)

```bash
alias wu='sudo wg-quick up wg0'
alias wd='sudo wg-quick down wg0'
alias ws='sudo wg show'
alias wl='watch -n 1 sudo wg show'
```

---

## Tips: Vis alle dine aliaser

```bash
alias               # List alle aktive aliaser
alias | grep git    # Finn git-aliaser
```

---

*Sist oppdatert: 2026 | archruud.org*
