//* ---- 💫 https://github.com/JaKooLit 💫 ---- *//
/* Waybar Modules Extras */

/* This is where you can add Extra Modules you wish. copy.sh will try to restore*/
/* this file along with the unique configs and styles. */
/* IMPORTANT, be mindful with , or ". else waybar will not start */

{
    "custom/menu#user": {
	"format": " 󰣇 ",
	"on-click": "killall rofi || $HOME/.config/rofi/launchers/type-2/launcher.sh",
	"tooltip": true,
	"tooltip-format": "Left Click: Rofi Menu",
    },

    "custom/wifiuser": {
	"format": "  ",
	"on-click": "nmgui",
	"tooltip": true,
	"tooltip-format": "Left Click: Wifi Menu",
    },

    "group/div#user": {
	    "orientation": "inherit",
	    "modules": [
			"custom/wifiuser",
		    "bluetooth",
			"pulseaudio",
		    "pulseaudio#microphone",
	    ]
    },

    "hyprland/workspaces#rw": {
    	"disable-scroll": true,
    	"all-outputs": false,
    	"warp-on-scroll": false,
    	"sort-by-number": true,
    	"show-special": false,
    	"on-click": "activate",
    	"on-scroll-up": "hyprctl dispatch workspace e+1",
    	"on-scroll-down": "hyprctl dispatch workspace e-1",
    	"persistent-workspaces": {
    		"*": 10
    	  },
    	"format": "{icon} {windows}",
    	"format-window-separator": " ",
    	"window-rewrite-default": " ",
    	"window-rewrite": {
    		"title<.*amazon.*>": " ",
    		"title<.*reddit.*>": " ",
    
    		"class<firefox|org.mozilla.firefox|librewolf|floorp|mercury-browser|[Cc]achy-browser>": " ",
    		"class<zen>": "󰰷 ",
    		"class<waterfox|waterfox-bin>": " ",
    		"class<microsoft-edge>": " ",
    		"class<Chromium|Thorium|[Cc]hrome>": " ",
    		"class<brave-browser>": "🦁 ",
    		"class<tor browser>": " ",
    		"class<firefox-developer-edition>": "🦊 ",
    
    		"class<kitty|konsole|[Aa]lacritty>": " ",
    		"class<kitty-dropterm>": " ",
    		"class<com.mitchellh.ghostty>": " ",
    		"class<org.wezfurlong.wezterm>": " ",
    		"class<Warp|warp|dev.warp.Warp|warp-terminal>": "󰰭 ",
    
    		"class<[Tt]hunderbird|[Tt]hunderbird-esr>": " ",
    		"class<eu.betterbird.Betterbird>": " ",
    		"title<.*gmail.*>": "󰊫 ",
    
    		"class<[Tt]elegram-desktop|org.telegram.desktop|io.github.tdesktop_x64.TDesktop>": " ",
    		"class<discord|discord-canary|[Ww]ebcord|[Vv]esktop|com.discordapp.Discord|dev.vencord.Vesktop>": " ",
    		"class<[Ss]ignal|signal-desktop|org.signal.Signal>": "󰍩 ",
    		"title<.*Signal.*>": "󰍩 ",
    		"title<.*whatsapp.*>": " ",
    		"title<.*zapzap.*>": " ",
    		"title<.*messenger.*>": " ",
    		"title<.*facebook.*>": " ",
    		"title<.*Discord.*>": " ",
    
    		"title<.*ChatGPT.*>": "󰚩 ",
    		"title<.*deepseek.*>": "󰚩 ",
    		"title<.*qwen.*>": "󰚩 ",
    		"class<subl>": "󰅳 ",
    		"class<slack>": " ",
    
    		"class<mpv>": " ",
    		"class<celluloid|Zoom>": " ",
    		"class<Cider>": "󰎆 ",
    		"title<.*Picture-in-Picture.*>": " ",
    		"title<.*youtube.*>": " ",
    		"class<vlc>": "󰕼 ",
    		"class<[Kk]denlive|org.kde.kdenlive>": "🎬 ",
    		"title<.*Kdenlive.*>": "🎬 ",
    		"title<.*cmus.*>": " ",
    		"class<[Ss]potify>": " ",
    		"class<Plex>": "󰚺 ",
    
    		"class<virt-manager>": " ",
    		"class<.virt-manager-wrapped>": " ",
    		"class<remote-viewer|virt-viewer>": " ",
    		"class<virtualbox manager>": "💽 ",
    		"title<virtualbox>": "💽 ",
    		"class<remmina|org.remmina.Remmina>": "🖥️ ",
    
    		"class<VSCode|code|code-url-handler|code-oss|codium|codium-url-handler|VSCodium>": "󰨞 ",
    		"class<dev.zed.Zed>": "󰵁",
    		"class<codeblocks>": "󰅩 ",
    		"title<.*github.*>": " ",
    		"class<mousepad>": " ",
    		"class<libreoffice-writer>": " ",
    		"class<libreoffice-startcenter>": "󰏆 ",
    		"class<libreoffice-calc>": " ",
    		"title<.*nvim ~.*>": " ",
    		"title<.*vim.*>": " ",
    		"title<.*nvim.*>": " ",
    		"title<.*figma.*>": " ",
    		"title<.*jira.*>": " ",
    		"class<jetbrains-idea>": " ",
    
    		"class<obs|com.obsproject.Studio>": " ",
    
    		"class<polkit-gnome-authentication-agent-1>": "󰒃 ",
    		"class<nwg-look>": " ",
    		"class<nwg-displays>": " ",
    		"class<[Pp]avucontrol|org.pulseaudio.pavucontrol>": "󱡫 ",
    		"class<steam>": " ",
    		"class<thunar|nemo>": "󰝰 ",
    		"class<Gparted>":"",
    		"class<gimp>": " ",
    		"class<emulator>": "📱 ",
    		"class<android-studio>": " ",
    		"class<org.pipewire.Helvum>": "󰓃",
    		"class<localsend>":"",
    		"class<PrusaSlicer|UltiMaker-Cura|OrcaSlicer>": "󰹛",
    
    		"class<io.github.kolunmi.Bazaar>": " ",
    		"title<^Bazaar$>": " ",
    
    		"class<com.gabm.satty>": " ",
    		"title<^satty$>": " ",
    
    		"class<[Bb]ox[Bb]uddy|io.github.dvlv.boxbuddy|io.github.dvlv.BoxBuddy>": " ",
    		"title<.*BoxBuddy.*>": " ",
    
    		"title<Hyprland Keybinds>": " ",
    		"title<Niri Keybinds>": " ",
    		"title<BSPWM Keybinds>": " ",
    		"title<DWM Keybinds>": " ",
    		"title<Emacs Leader Keybinds>": " ",
    		"title<Kitty Configuration>": " ",
    		"title<WezTerm Configuration>": " ",
    		"title<Yazi Configuration>": " ",
    		"title<Cheatsheets Viewer>": " ",
    		"title<Documentation Viewer>": " ",
    		"title<^Wallpapers$>": " ",
    		"title<^Video Wallpapers$>": " ",
    		"title<^qs-wlogout$>": " ",
    		}
    	},
}
