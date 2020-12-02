# What's this?

Hi, I'm `pport` an this are my dotfiles. 

I'm still studying and there are many times I need Windows, because of programs that doesn't work in a VM. For that reason and also learning *NIX (by distro-hopping), I wipe my hard drive frequently. This is practical guide to quickly reinstall my favorite distro (Arch Linux) and also to keep track of modifications in my system.

If you have fallen in this repo and you see an error, feel free to PM me or open an issue. Thanks!

[TOC]

# To-Do

- [ ] Nvidia drivers (prime render off-load method)
- [ ] PowerTOP management
- [ ] Picom config
- [ ] Dunst config
- [ ] Base16 management

# Programs

| Use                   | Name                  | Description |
| ----                  | ----                  | ---- |
| Graphical Server      | xorg                  | FOSS X server protocol implementation |
| Backlight             | xbacklight            | |
| Sound                 | pulseaudio            | |
| WM                    | bspwm                 | binary-space partition window manager |
| HotKey Daemon         | sxhkd                 | Simple X hotkey daemon |
| Compositor            | picom                 |  |
| Notifications         | dunst                 |  |
| Bar                   | polybar               | |
| Font 1                | Fira Code Nerd Font   | |
| Font 2 (only icons)   | MaterialDesignIcons | |
| Screenshot            | scrot                 | |

# Configs

## Only for XPS 15 7590 ([ArchWiki](https://wiki.archlinux.org/index.php/Dell_XPS_15_7590))

### Default sleep mode
Add ``mem_sleep_default=deep`` to kernel parameters and rebuild iniramfs image.

## Autologin ([ArchWiki](https://wiki.archlinux.org/index.php/Getty#Automatic_login_to_virtual_console))
Execute this command and add the next to the file:

```bash
# systemctl edit getty@tty1.service.d/override.conf
```

```ini
[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --autologin uSeRnAmE --noclear %I $TERM
```

## Silent Boot ([ArchWiki](https://wiki.archlinux.org/index.php/Silent_boot))

### Remove kernel messages
Add the next kernel parameters to the bootloader:
```
quiet loglevel=3 rd.systemd.show_status=auto rd.udev.log_priority=3
```
Rebuild initramfs.

### Remove console cursor blinking
Add ``vt.global_cursor_default=0`` to the kernel parameters.
To recover the cursor in the TTY, run:
```bash
sudo -i
setterm -cursor on >> /etc/issue
```

### Remove last login message
```bash
touch ~./hushlogin
```

### Remove fsck messages

Replace ``udev`` with ``systemd`` hook in ``/etc/mkinitcpio.conf`` and update the initramfs image:

```bash
# mkinitcpio -p PRESET
```

> INFO: If encryption is in use with ``systemd`` hook, see [this](https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration#mkinitcpio).

Edit this two services:
```bash
# systemctl edit --full systemd-fsck-root.service
# systemctl edit --full systemd-fsck@.service
```
And configure *StandardOutput* and *StandardError* like this:
```ini
(...)

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/systemd/systemd-fsck
StandardOutput=null
StandardError=journal+console
TimeoutSec=0
```

### Remove getty message (autologin)
Add or modify the override configuration with ``--skip-login --nonewline --noissue``

```bash
# systemctl edit getty@tty1
```

```ini
[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --skip-login --nonewline --noissue --autologin uSeRnAmE --noclear %I $TERM
```

## Locale configuration ([ArchWiki](https://wiki.archlinux.org/index.php/Locale))
### Generating locale
Uncomment locales in ``/etc/locale.gen``
```bash
...
#en_SG ISO-8859-1
en_US.UTF-8 UTF-8
#en_US ISO-8859-1
...
```
Save the file and generate the locales:

```bash
# locale-gen
```

### Setting locale (system-wide)
Edit and populate the file `/etc/locale.conf`:
```bash
LANG=en_US.UTF-8
LC_MESSAGES=en_US.UTF-8
LC_TIME=es_ES.UTF-8
LC_COLLATE=C
```
### Setting locale (per-user)

Alternatively, you can set the locale in `~/.config/locale.conf` the same way as system-wide.

> INFO: this is how i'm doing right now.

## Login Keyring ([ArchWiki](https://wiki.archlinux.org/index.php/GNOME/Keyring#Using_the_keyring_outside_GNOME))

Install `gnome-keyring` and `libsecret` package. Optionally `seahorse` for a GUI interface.

If you are using autologin, set a blank password to the default `Login`. But first you need to disable autologin to create the default `Login` keyring. Once it's created, set no password to achieve auto-unlocking.

> WARNING: this leaves the passwords unencrypted. I don't care much because I have full disk encryption. Nevertheless I think it can be obtained by software digging into the filesystem.

Also, add this to `~/.xinitrc` (in my case `bspwmrc`) to autostart the daemon:

```bash
eval $(/usr/bin/gnome-keyring-daemon --start --components=pkcs11,secrets,ssh)
export SSH_AUTH_SOCK
```

> WARNING: There is a bug in my config or the ArchWiki isn't correct, this solved it: https://bbs.archlinux.org/viewtopic.php?id=224652