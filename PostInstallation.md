# Post installation steps

##### 1. Users (<https://wiki.archlinux.org/index.php/users_and_groups>)

Add non-priviliged user for daily use and set password:
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

- install `openssh` and enable service:
```
# systemctl enable sshd.service
```

- enable root user to login (**DON'T DO THAT!**) granting those lines in `/etc/ssh/sshd_config` are not commented:
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

- install `gnome`, and enable the display manager service `gdm.service`

- install `gnome-tweaks` for advanced settings, and `gnome-shell-extension-multi-monitors-add-on-git` (from `AUR`) for advanced multi-monitor support

- enable fractional re-scaling for huge resolutions:
```
$ gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

- to enable network management from GNOME control panel install `networkmanager` and enable the service `NetworkManager.service`

- as above, for bluetooth, install `bluez` and `bluez-utils`, and then enable the service `bluetooth.service`

##### 6. Automatic backup with `rsync` (<https://wiki.archlinux.org/index.php/Rsync>) and timers (<https://wiki.archlinux.org/index.php/Systemd/Timers>)

- `sync.sh` bash script for **differential** backup with `rsync`:
```
#!/bin/sh

logFile="log_"`date +%Y%m%d%H%M%S`".log"

echo "Start on "`date` >> $logFile

rsync -avzh --delete \
	/path/to/source \
	/path/to/destination \
	>> $logFile

echo "End on "`date` >> $logFile
```

- `/etc/systemd/system/data-backup.service`:
```
[Unit]
Description=Trigger rsync script to backup data

[Service]
Type=oneshot
User=andrea
ExecStart=/bin/bash /path/to/sync.sh

[Install]
WantedBy=multi-user.target
```

- setup a timer creating `/etc/systemd/system/data-backup.timer`:
```
[Unit]
Description=Do backup on boot, then once a day

[Timer]
OnBootSec=15min
OnUnitActiveSec=24h 

[Install]
WantedBy=timers.target
```

- enable the service `data-backup.timer`

##### 7. Nvidia GPU configuration (<https://wiki.archlinux.org/index.php/NVIDIA>)

- disable `Wayland` uncommenting `#WaylandEnable=false` line in `/etc/gdm/custom.conf`
```
[daemon]
WaylandEnable=false
```

- install nvidia drivers
```
# pacman -S nvidia
```

- disable intel (because of double video cards) and nouveau modules:
```
# echo install i915 /usr/bin/false >> /etc/modprobe.d/blacklist.conf
# echo install intel_agp /usr/bin/false >> /etc/modprobe.d/blacklist.conf
# echo blacklist nouveau > /etc/modprobe.d/nonouveau.conf
```

- test hw acceleration for HEVC:
```
$ mpv --hwdec=nvdec --vo=gpu path/to/video
```
