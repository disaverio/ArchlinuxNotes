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
CXXFLAGS="${CFLAGS}"
```

- since `make` uses `MAKEFLAGS` env var, we can set there the number of jobs to run simultaneously. Add `export MAKEFLAGS="-j$(nproc)"` to `/etc/profile` <-- if some build fails it could be because of race conditions. Remove it.

- install `yay` as `AUR` helper: [download snapshot from AUR](https://aur.archlinux.org/packages/yay), extract it, `cd` in folder and:
```
# makepkg -si 
```

##### 5. GNOME Desktop Environment (<https://wiki.archlinux.org/index.php/GNOME>)

- install `gnome`, and enable the display manager service `gdm.service`

- install `gnome-tweaks` for advanced settings, and `gnome-shell-extension-multi-monitors-add-on-git` (from `AUR`) for advanced multi-monitor support

- enable fractional re-scaling for [huge resolutions](https://wiki.archlinux.org/index.php/HiDPI) (**in Wayland**):
```
$ gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

- enable fractional re-scaling for [huge resolutions](https://wiki.archlinux.org/index.php/HiDPI) (**in XORG**):

install `xrandr`, then double the re-scaling value, then set correct values for two monitors:
```
$ gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "[{'Gdk/WindowScalingFactor', <2>}]"
$ gsettings set org.gnome.desktop.interface scaling-factor 2
$ xrandr --output DVI-D-0 --scale 2x2 --pos 0x100 --output DP-0 --scale 1.5x1.5 --pos 3840x0
```
to keep changes after reboot:
```
$ echo "/usr/bin/xrandr --output DVI-D-0 --scale 2x2 --pos 0x100 --output DP-0 --scale 1.5x1.5 --pos 3840x0" >> ~/.xprofile
$ chmod +x ~/.xprofile
```

- to enable network management from GNOME control panel install `networkmanager` and enable the service `NetworkManager.service`

- as above, for bluetooth, install `bluez` and `bluez-utils`, and then enable the service `bluetooth.service`

- in case of multiple monitors, if gdm login screen starts on secondary monitory copy monitor configuration to gdm, and make it readable to gdm user:
```
$ sudo cp -f ~/.config/monitors.xml ~gdm/.config/monitors.xml
# chown $(id -u gdm):$(id -g gdm) ~gdm/.config/monitors.xml
```

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

##### 8. Hardware video acceleration (<https://wiki.archlinux.org/index.php/Hardware_video_acceleration>)

- Since `nvidia` drivers are installed, hw video acceleration should already be working on NVDECODE/NVENCODE backend. test it for HEVC:
```
$ mpv --hwdec=nvdec --vo=gpu path/to/video
```

Not every software is able to leverage NVDECODE/NVENCODE so:

- enable VDPAU installing (if not yet installed by `nvidia` as dependency) `libvdpau` and `vdpauinfo` packages, and check it running `vdpauinfo`

- enable VA-API support installing `libva` and `libva-vdpau-driver` packages. Then check it's working running `vainfo` (provided by `libva-utils` package). VA-API uses VDPAU as backend in nvidia environments.

- to have chromium with hw video acceleration install `libva-vdpau-driver-chromium` (it will remove `libva-vdpau-driver` because conflicting) and `chromium-vaapi-bin` (chomium patched to support vaapi). then enable flags with:
```
$ echo --ignore-gpu-blacklist > ~/.config/chromium-flags.conf
$ echo --enable-gpu-rasterization >> ~/.config/chromium-flags.conf
$ echo --enable-native-gpu-memory-buffers >> ~/.config/chromium-flags.conf
$ echo --enable-zero-copy >> ~/.config/chromium-flags.conf
```

##### 9. Others

Gists:
- [.bashrc for customized prompt with time and colored git](https://gist.github.com/disaverio/dd6929cb1ae5873dcf5675ee83311451)
- [cutomized behaviour for Alt+Tab scoped to current workspace](https://gist.github.com/disaverio/4e53806a736764bcd571fca7643e4c34)
