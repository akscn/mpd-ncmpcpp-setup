# 🎵 mpd + ncmpcpp Setup

> Terminal music player setup for **Arch Linux** with Hyprland.  
> Uses mpd as the music daemon and ncmpcpp as the TUI frontend, with a spectrum visualizer.  
> Covers both a **generic Hyprland** setup and **end4's dotfiles** specifically — differences are clearly marked.

---

## Table of Contents

- [What is this stack?](#what-is-this-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
  - [mpd config](#1-mpd-config)
  - [ncmpcpp config](#2-ncmpcpp-config)
  - [Hyprland keybind — end4 dotfiles](#3-hyprland-keybind--end4-dotfiles)
  - [Window rules — end4 dotfiles](#4-window-rules--end4-dotfiles)
  - [Hyprland keybind — Generic](#5-hyprland-keybind--generic)
  - [Window rules — Generic](#6-window-rules--generic)
- [Starting mpd](#starting-mpd)
- [Adding Music & Launching](#adding-music--launching)
- [Key Shortcuts](#key-shortcuts)
- [Gotchas & Fixes](#gotchas--fixes)
- [File Structure](#file-structure)

---

## What is this stack?

| Tool | Role |
|------|------|
| `mpd` | Music Player Daemon — runs in the background, manages your library |
| `ncmpcpp` | TUI client for mpd — the interface you actually use |
| `mpc` | CLI tool to send quick commands to mpd (update library, etc.) |
| `ueberzugpp` | Image rendering in terminal (for album art, optional) |

mpd and ncmpcpp are separate. mpd plays music silently in the background; ncmpcpp is just one way to control it.

---

## Prerequisites

- Arch Linux
- PipeWire (for audio output)
- Hyprland (generic or end4's dotfiles)
- Fish shell (affects some commands — noted where relevant)

---

## Installation

```bash
sudo pacman -S mpd ncmpcpp mpc
yay -S ueberzugpp       # optional, for album art
```

---

## Configuration

### 1. mpd config

Create the required directories first:

```bash
mkdir -p ~/.config/mpd
mkdir -p ~/.config/ncmpcpp/lyrics
mkdir -p ~/.local/share/mpd
mkdir -p ~/Music
```

> ⚠️ Do NOT manually create the database file with `touch`. mpd creates it itself on first run.

Create the config:

```bash
nano ~/.config/mpd/mpd.conf
```

Paste this:

```
music_directory     "~/Music"
db_file             "~/.local/share/mpd/database"
log_file            "~/.local/share/mpd/log"
pid_file            "~/.local/share/mpd/pid"
state_file          "~/.local/share/mpd/state"
bind_to_address     "127.0.0.1"
port                "6600"
auto_update         "yes"

audio_output {
    type    "pipewire"
    name    "PipeWire Output"
}

audio_output {
    type            "fifo"
    name            "Visualizer"
    path            "/tmp/mpd.fifo"
    format          "44100:16:2"
}
```

> The FIFO audio output is what feeds the spectrum visualizer in ncmpcpp. Both outputs run simultaneously.

---

### 2. ncmpcpp config

```bash
nano ~/.config/ncmpcpp/config
```

Paste this:

```
## Connection
mpd_host                            = "127.0.0.1"
mpd_port                            = "6600"
mpd_connection_timeout              = "5"
mpd_crossfade_time                  = "3"

## Paths
ncmpcpp_directory                   = "~/.config/ncmpcpp"
lyrics_directory                    = "~/.config/ncmpcpp/lyrics"

## UI
user_interface                      = "alternative"
alternative_header_first_line_format  = "$b$6{%t}|{%f}$/b$9 $1─ $7{%a}$9"
alternative_header_second_line_format = "$5{%b}$9 $1─ $3{%y}$9"
now_playing_prefix                  = "$b$2▶ "
now_playing_suffix                  = "$/b"
song_list_format                    = "{$7%a - $9}{%t}|{%f}$R$4%l$9"
song_columns_list_format            = "(6f)[magenta]{l} (40)[cyan]{t|f:Title} (30)[blue]{a} (24)[green]{b}"
song_status_format                  = "$b%t$/b $2by$9 $3%a$9 $2from$9 $4%b$9"

## Colors
colors_enabled                      = "yes"
empty_tag_color                     = "magenta"
header_window_color                 = "cyan"
volume_color                        = "green"
state_line_color                    = "default"
state_flags_color                   = "default:b"
main_window_color                   = "default"
color1                              = "cyan"
color2                              = "magenta"
progressbar_color                   = "black:b"
progressbar_elapsed_color           = "cyan:b"
statusbar_color                     = "default"
statusbar_time_color                = "cyan:b"

## Progress bar
progressbar_look                    = "━━╸"

## Visualizer
visualizer_data_source              = "/tmp/mpd.fifo"
visualizer_output_name              = "Visualizer"
visualizer_in_stereo                = "yes"
visualizer_type                     = "spectrum"
visualizer_look                     = "▋▋"
visualizer_color                    = "cyan,magenta,blue,green"
visualizer_fps                      = "60"
visualizer_autoscale                = "yes"

## Misc
follow_now_playing_lyrics           = "yes"
fetch_lyrics_for_current_song_in_background = "yes"
seek_time                           = "1"
playlist_display_mode               = "columns"
browser_display_mode                = "columns"
search_engine_display_mode          = "columns"
incremental_seeking                 = "yes"
show_hidden_files_in_local_browser  = "no"
default_find_mode                   = "wrapped"
clock_display_seconds               = "yes"
display_volume_level                = "yes"
display_bitrate                     = "yes"
mouse_support                       = "yes"
```

---

### 3. Hyprland keybind — end4 dotfiles

> Keybinds file is at `~/.config/hypr/hyprland/keybinds.conf` in end4's dotfiles, not the default `~/.config/hypr/hyprland.conf`.

Check what's already taken before adding:

```bash
grep -i "Ctrl.*M\|M.*Ctrl" ~/.config/hypr/hyprland/keybinds.conf
```

If nothing comes back, add the keybind:

```bash
echo 'bindd = Ctrl+Super, M, Open ncmpcpp music player, exec, kitty --class ncmpcpp -e ncmpcpp' >> ~/.config/hypr/hyprland/keybinds.conf
```

> `Super+M` is taken by media controls and `Super+Shift+M` is mute in end4's config. `Ctrl+Super+M` is free.

---

### 4. Window rules — end4 dotfiles

> Rules file is at `~/.config/hypr/custom/rules.conf` in end4's dotfiles.

> ⚠️ end4 uses newer Hyprland which uses `windowrule` NOT `windowrulev2`. Argument order is also different.

Open the file:

```bash
nvim ~/.config/hypr/custom/rules.conf
```

Add at the bottom (press `G` to go to end, `o` for new line):

```
windowrule = float on, match:class ^(ncmpcpp)$
windowrule = center 1, match:class ^(ncmpcpp)$
windowrule = size 900 600, match:class ^(ncmpcpp)$
```

Save with `:wq`.

> ⚠️ **Fish shell note:** Do NOT use `<<EOF` heredoc syntax in Fish — it doesn't work. Always edit files directly with nvim or use `printf` to append.

Reload Hyprland:

```bash
hyprctl reload
```

---

---

### 5. Hyprland keybind — Generic

> For a standard Hyprland setup where everything lives in `~/.config/hypr/hyprland.conf` or a sourced binds file.

Check if `Ctrl+Super+M` is free:

```bash
grep -i "Ctrl.*M\|M.*Ctrl" ~/.config/hypr/hyprland.conf
```

If nothing comes back, add the keybind:

```bash
echo 'bind = CTRL SUPER, M, exec, kitty --class ncmpcpp -e ncmpcpp' >> ~/.config/hypr/hyprland.conf
```

---

### 6. Window rules — Generic

> Generic Hyprland still uses `windowrulev2` — it's only end4's newer build that deprecated it.

Open your config:

```bash
nvim ~/.config/hypr/hyprland.conf
```

Add at the bottom:

```
windowrulev2 = float, class:^(ncmpcpp)$
windowrulev2 = size 900 600, class:^(ncmpcpp)$
windowrulev2 = center, class:^(ncmpcpp)$
```

Reload Hyprland:

```bash
hyprctl reload
```

---

## Starting mpd

Enable and start mpd as a user service:

```bash
systemctl --user enable --now mpd
systemctl --user status mpd
```

You should see `active (running)`. The only expected warning is:

```
decoder: Decoder plugin "wildmidi" is unavailable
```

This is harmless — it just means MIDI support isn't configured. Normal music files are unaffected.

---

## Adding Music & Launching

Put your music files in `~/Music`, then:

```bash
mpc update       # scans ~/Music and updates the mpd database
ncmpcpp          # launch the TUI
```

Or press `Ctrl+Super+M` to open it in a floating window via Hyprland.

---

## Key Shortcuts

| Key | Action |
|-----|--------|
| `2` | File browser |
| `space` | Add song to playlist |
| `enter` | Play selected |
| `p` | Pause / Resume |
| `>` / `<` | Next / Previous track |
| `+` / `-` | Volume up / down |
| `` ` `` (backtick) | Switch to visualizer |
| `r` | Toggle repeat |
| `z` | Toggle shuffle |
| `q` | Quit |

---

## Gotchas & Fixes

### `unrecognized parameter: "sticker_database"`

Some config templates include `sticker_database` but not all mpd versions support it. Remove it:

```bash
sed -i '/sticker_database/d' ~/.config/mpd/mpd.conf
```

---

### `exception: Database corrupted` or `Failed to open database: No such file or directory`

Never create the database file manually. If you did, delete it and let mpd recreate it:

```bash
rm ~/.local/share/mpd/database
systemctl --user restart mpd
```

---

### `windowrulev2` deprecated warning in Hyprland

Newer Hyprland (used by end4's dotfiles) deprecated `windowrulev2`. Use `windowrule` with this syntax instead:

```
# Old (broken)
windowrulev2 = float, class:^(ncmpcpp)$

# New (correct)
windowrule = float on, match:class ^(ncmpcpp)$
windowrule = center 1, match:class ^(ncmpcpp)$
windowrule = size 900 600, match:class ^(ncmpcpp)$
```

---

### `fish: Expected a string, but found a redirection` with `<<EOF`

Fish shell doesn't support bash heredoc (`<<EOF`) syntax. Use `printf` or edit files directly in nvim instead.

---

## File Structure

```
~/.config/
├── mpd/
│   └── mpd.conf               # mpd configuration
├── ncmpcpp/
│   ├── config                 # ncmpcpp configuration
│   └── lyrics/                # auto-fetched lyrics stored here
└── hypr/
    ├── hyprland/
    │   └── keybinds.conf      # Ctrl+Super+M keybind added here
    └── custom/
        └── rules.conf         # ncmpcpp window rules added here

~/.local/share/
└── mpd/
    ├── database               # auto-created by mpd, don't touch
    ├── log
    ├── pid
    └── state

~/Music/                       # put your music files here
```
