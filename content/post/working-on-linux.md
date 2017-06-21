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
    3. Run command as `sudo dd if=~/Downloads/manjaro-kde-17.0.1-stable-x86_64.iso of=/dev/sdc` 
    (You may replace the ISO file name and target device name by yourself). This will take even over 10 minutes. 
    For Windows user, try software like *Fedora media writer*, *rawrite* and *UltraISO*.
2. Restart your PC and press `F12` to choose boot from *USB flash driver*.
3. Start Installation. Choose correct partition table type(`GPT` for UEFI supported motherboard, 
with manually choose UEFI boot later. `MBR` for others). 
You never need set a mount point of `/home` at this step, 
for it should be mounted with another hard disk who will storage data. 
Volume of these partitions should be set as below:

    | Directories | Volume     | File system |
    | ----------- | ---------- | ----------- |
    | /boot       | 512M [^1]  | ext4        |
    | swap        | 2048M [^2] | Linux-swap  |
    | /           | remaining space [^3]     | ext4        |

4. After installation, reboot and try mount `/home`.
    1. Use `sudo fdisk -l` to check all mountable devices.
    2. Make partition: **Suppose another hard disk is recognized as `/dev/sdb`** (for example). 
    If its volume less than 2TB, you can simply use `fdisk` to make partition table for it. 
    Else, you may use another advanced tool like `parted`. 
    Run `parted /dev/sdb` to start, you will get a `(parted)` prompt.
    Here we want to create only one partition, whose type is `primary` with `ext4` file system, 
    just type `mkpart primary ext4 0% 100%`.

    {{% alert note %}}
Some users may notice that `print` can be used to show information of current device, 
then they may use `mkpart primary ext4 0 8TB` (for example) to build partition table, 
which sometimes raise a warning message:

Warning: The resulting partition is not properly aligned for best performance.

Ignore/Cancel?

The warning means the partition start is not aligned. Anyway, `0KB` is not real start of your disk.
    {{% /alert %}}

5. Run `mkfs.ext4 -F /dev/sdb1` to reformat partition (`parted`'s `ext4` parameter doesn't works, I don't know why yet).
6. `mount /dev/sdb1 /home` to mount new disk as `/home`. To make this mount point auto-mounted, 
add `/dev/sdb1  /home  ext4  defaults,noatime  0  2` at the last line of `/etc/fstab`, then restart.
7. To configure auto-starting services with `systemd` (use the `sshd.service` as example, 
you may also add `teamviewerd` and `rstudio-server.service` for auto-start them):
    1. Use the command like `sudo systemctl enable sshd.service` to enable the service, 
    and this should create a symlink in `/etc/systemd/system/multi-user.target.wants/` that looks like the following 
    (do **NOT** create this manually):
    
        ```pre
        lrwxrwxrwx 1 root root 38 Aug  1 04:43 /etc/systemd/system/multi-user.target.wants/service.service -> /usr/lib/systemd/system/service.service
        ```
    
    2. `vim /etc/systemd/system/multi-user.target.wants/sshd.service` to check the content of the service file. 
    Make sure the file contains a line like `Restart=always` under the `[Service]` section of the file to 
    enable the service to respawn after a crash.
    3. Finally, reload the `systemd daemon`, followed by a restart of the service:
    
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl restart sshd.service
        ```
    
8. If you have /home directory on KDE platform before, 
you may run `cd; rm -rf .cache .config .kde4 .local` to reset your KDE environment.

## Autostart
To make programs (Yakuake, etc.) autostart:
Alt + F2 -> autostart -> Add Program...

## R!E!P!O!
```bash
sudo pacman-mirrors -g
sudo pacman-optimize && sync
sudo pacman -Syyu
```
For some Chinese users, generating mirror list may raise an error (probably caused by network), 
you can use `sudo pacman-mirrors -g -c china` instead.

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

## VI && VIM
`sudo pacman -S vim`

## Set users as sudoers
By typing `sudo visudo`, you can change sudoers in *vi* mode, 
then use `/root` with `n` (perhaps more than one time) 
to change the position of cursor to the `User privilege specification` section. 
Press `yyp` to copy & paste the current line, then `dft` to delete username 'root', 
finally add the new sudoer's username. Press `:wq` to save and exit.
Now this section of configuration file seems like:
```pre
##
## User privilege specification
##
root ALL=(ALL) ALL
ysun ALL=(ALL) ALL
```

{{% alert note %}}
If you want add a sudoer who never need password to run commands as root, you can set as:
```pre
ysun        ALL=(ALL)     NOPASSWD:ALL
```
{{% /alert %}}

## TeamViewer
`sudo pacman -S teamviewer`

## gcc-fortran
`sudo pacman -S gcc-fortran`

## locate
```bash
sudo pacman -S mlocate
sudo updatedb
```

## Tk

`sudo pacman -S tk`

## Atom
`sudo pacman -S atom`

## ibus-rime
`sudo pacman -S ibus-rime`

## shadowsocks
`sudo pacman -S shadowsocks-qt5`

## R && RStudio
`sudo pacman -S r`

`sudo yaourt -S rstudio-desktop-bin`

For RStudio server release, run `sudo yaourt -S rstudio-desktop-bin` instead.

## Axel
`sudo pacman -S axel`

## Steam
Steam outputs this error and exits.
```pre
symbol lookup error: /usr/lib/libxcb-dri3.so.0: undefined symbol: xcb_send_request_with_fds
```
For steam to work, disable dri3 in xorg config file or as a workaround run steam with `LIBGL_DRI3_DISABLE=1`.
`LIBGL_DRI3_DISABLE=1 steam`

## QQ (Do **NOT** install this on server)
`yaourt crossover`

## Virtualbox
See [*manjaro wiki*](https://wiki.manjaro.org/index.php?title=Virtualbox) here.

[^1]: Need 190MB at least for *Fedora*, may even less for *Arch Linux*.
[^2]: As a server for bioinformatics use, we should use RAM first, so there's no need of too much memory of `swap`.
[^3]: As the disk maintaining operation system, `/` should keep all remaining space.
