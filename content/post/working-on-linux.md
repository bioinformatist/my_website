+++
image = ""
date = "2017-04-20T20:05:36+08:00"
title = "Working with Manjaro KDE Linux"
tags = ['Linux']
highlight = true
math = false
summary = "Guide for say Goodbye to Windows"

+++

## OS Installation
1. Preparation of a USB media.
 1. Download the proper [*Manjaro KDE Edition*](https://manjaro.org/get-manjaro/) disc image (ISO file).
 2. Check your device list using `sudo fdisk -l`.
 3. Run command as `sudo dd if=~/Downloads/manjaro-kde-17.0.1-stable-x86_64.iso of=/dev/sdc` (You may replace the ISO file name and target device name by yourself). This will take even over 10 minutes. For Windows user, try software like *Fedora media writer*, *rawrite* and *UltraISO".
2. Restart your PC and press `F12` to choose boot from *USB flash driver*.
3. Start Installation. **TODO: /home, MBR and GPT**

| Directories | Volume     | File system |
| ----------- | ---------- | ----------- |
| /boot       | 512M [^1]  | ext4        |
| swap        | 2048M [^2] | Linux-swap  |
| /           | - [^3]     | ext4        |

3. **TODO: /home, /etc/fstab**
4. If you have /home directory on KDE platform before, you may run `cd; rm -rf .cache .config .kde4 .local` to reset your KDE environment.

## Autostart
To make programs (Yakuake, etc.) autostart:
Alt + F2 -> autostart -> Add Program...

## R!E!P!O!
```bash
sudo pacman-mirrors -g
sudo pacman-optimize && sync
sudo pacman -Syyu
```

`sudo nano /etc/pacman.conf`

```pre
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

```bash
sudo pacman -Syyu
sudo pacman -S archlinuxcn-keyring
```

## Atom
`sudo pacman -S atom`

## ibus-rime
`sudo pacman -S ibus-rime`

## shadowsocks
`sudo pacman -S shadowsocks-qt5`

## R && Rstudio
`sudo pacman -S r`
`sudo yaourt -S rstudio-desktop-bin`

## Steam
Steam outputs this error and exits.
```pre
symbol lookup error: /usr/lib/libxcb-dri3.so.0: undefined symbol: xcb_send_request_with_fds
```
For steam to work, disable dri3 in xorg config file or as a workaround run steam with `LIBGL_DRI3_DISABLE=1`.
`LIBGL_DRI3_DISABLE=1 steam`

## QQ
`yaourt crossover`

[^1]: Need 190MB at least for *Fedora*, may even less for *Arch Linux*.
[^2]: **TODO**
[^3]: **TODO**
