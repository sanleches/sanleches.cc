---
title: Seven Lines of Bash to Fix My Boring GNOME Wallpaper
date: 2026-08-06 12:00:00
category: projects
tags:
  - Linux
  - GNOME
  - Bash
  - desktop
---

I had a folder full of wallpapers and GNOME kept showing one of them. This felt like a problem that deserved somewhere between five and ten lines of Bash.

[gnome-wallpaper-randomizer](https://github.com/sanleches/gnome-wallpaper-randomizer) picks a random image from `~/Pictures/Wallpapers` and applies it when I log in. There is no daemon, extension, scheduler, service, or Electron application quietly consuming half a gigabyte of RAM. GNOME autostart runs the script once and the script exits.

The important part is basically this:

```bash
PIC=$(find "$HOME/Pictures/Wallpapers" -maxdepth 1 -type f \
  \( -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.png' \
     -o -iname '*.bmp' -o -iname '*.webp' \) | shuf -n1)

gsettings set org.gnome.desktop.background picture-uri-dark "file://$PIC"
gsettings set org.gnome.desktop.background picture-uri "file://$PIC"
```

Both GSettings keys matter. GNOME has separate wallpaper URIs for dark and normal appearance modes, so changing only one can make the wallpaper jump back when the theme changes.

## Making it start itself

The installer copies the script to `~/.local/bin` and generates an XDG autostart entry under `~/.config/autostart`. The repository contains a desktop-entry template with a placeholder path; `install.sh` replaces that placeholder with the absolute installed path.

The absolute path is intentional. A desktop entry does not treat `~` like a shell does, so this is unreliable:

```ini
Exec=~/.local/bin/random-wallpaper.sh
```

Generating the real path avoids that tiny login-time mystery.

## It is small because the requirements are small

The script looks only at regular files in the top level of the wallpaper directory. It does not remember the last selection, avoid repeats, recurse through albums, or rotate on a timer. If the folder is empty, it silently does nothing.

Those would all be reasonable features, but I like that the current program is easy to inspect in one glance. `find` discovers images, `shuf` chooses one, and `gsettings` tells GNOME about it. The operating system already has all the pieces; the project just connects them.

Code: [github.com/sanleches/gnome-wallpaper-randomizer](https://github.com/sanleches/gnome-wallpaper-randomizer)
