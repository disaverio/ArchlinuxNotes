# Reccomended steps for a minimal fresh archlinux installation

**1.** Add non-priviliged user for daily use and set password:
```
# useradd -m username
# passwd username
```

##### 2. SUDO (<https://wiki.archlinux.org/index.php/Sudo>)

Install `sudo` to the system, and grant new user adding `username ALL=(ALL) ALL` to conf file:
```
# EDITOR=nano visudo
```

##### 3. SSH (<https://wiki.archlinux.org/index.php/OpenSSH>)

Install `openssh` and enable root user to login (**DON'T DO THAT!**) checking those lines in `/etc/ssh/sshd_config` are not commented:
```
PermitRootLogin yes
PasswordAuthentication yes
```

##### 4. AUR (<https://wiki.archlinux.org/index.php/Arch_User_Repository>)

- install needed for building your own packages
```
# pacman -S --needed base-devel
```

- adjust `/etc/makepkg.conf` to optimize your compilation process. Set:
```
CFLAGS="-march=native -O2 -pipe -fno-plt"
CXXFLAGSCXXFLAGS="${CFLAGS}"
```

- since `make` uses `MAKEFLAGS` env var, we can set there the number of jobs to run simultaneously. Add `export MAKEFLAGS="-j$(nproc)"` to `/etc/profile` <-- if some build fails it could be because of race conditions. Remove it.

- install `yay` as `AUR` helper: [download snapshot from AUR](https://aur.archlinux.org/packages/yay), extract it, `cd` in folder and:
```
# makepkg -si 
```

##### 5. GNOME Desktop Environment (<https://wiki.archlinux.org/index.php/GNOME>)

- install `gnome`. It includes `gdm`, so enable it:
```
# systemctl enable gdm.service
```

- install `gnome-tweaks` for advanced settings, and `gnome-shell-extension-multi-monitors-add-on-git` (from `AUR`) for advanced multi-monitor support

- enable fractional re-scaling for huge resolutions:
```
$ gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

- to enable network management from GNOME control panel install `networkmanager` and enable the service `NetworkManager.service`

- as above, for bluetooth, install `bluez` and `bluez-utils`, and then enable the service `bluetooth.service`