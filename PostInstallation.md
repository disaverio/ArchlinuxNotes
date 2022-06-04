# Post installation steps

##### 1. Users (<https://wiki.archlinux.org/index.php/users_and_groups>)

Add non-priviliged user for daily use and set password:
```
# useradd -m username
# passwd username
```

##### 2. Sudo (<https://wiki.archlinux.org/index.php/Sudo>)

Install `sudo` to the system, and grant new user by adding `username ALL=(ALL) ALL` to conf file:
```
# EDITOR=nano visudo
```
Also remove password waiting timeout by adding `Defaults passwd_timeout=0`

##### 3. SSH (<https://wiki.archlinux.org/index.php/OpenSSH>)

- to continue installation process from another system, install `openssh` and start the service:
```
# systemctl start sshd.service
```
- enable root user to login by uncommenting those lines in `/etc/ssh/sshd_config`:
```
PermitRootLogin yes
PasswordAuthentication yes
```
- don't forget to remove above lines when completed

##### 4. AUR (<https://wiki.archlinux.org/index.php/Arch_User_Repository>)

- install the needed packages to build your owns:
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
- install `gnome-tweaks` for advanced settings
- install `gnome-shell-extension-unite` to customize gnome
- to enable network management from GNOME control panel install `networkmanager`, `openvpn`, `networkmanager-openvpn` and enable the service `NetworkManager.service`
- to enable bluetooth, install `bluez` and `bluez-utils`, enable the service `bluetooth.service`, and uncomment the line `AutoEnable=true` in `/etc/bluetooth/main.conf`
- in case of multiple monitors, if gdm login screen starts on secondary monitory, copy monitor configuration to gdm, and make it readable to gdm user:
```
$ sudo cp -f ~/.config/monitors.xml ~gdm/.config/monitors.xml
# chown $(id -u gdm):$(id -g gdm) ~gdm/.config/monitors.xml
```

In case of monitor with [hi resolutions](https://wiki.archlinux.org/index.php/HiDPI) (>=4k) you may need fractional rescaling:

- enable it in Wayland:
```
$ gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```
- enable it in Xorg:
  - install `xrandr`, then double the re-scaling value, then set correct values for two monitors:
  ```
  $ gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "[{'Gdk/WindowScalingFactor', <2>}]"
  $ gsettings set org.gnome.desktop.interface scaling-factor 2
  $ xrandr --output DVI-D-0 --scale 2x2 --pos 0x100 --output DP-0 --scale 1.5x1.5 --pos 3840x0
  ```
  - to keep changes after reboot:
  ```
  $ echo "/usr/bin/xrandr --output DVI-D-0 --scale 2x2 --pos 0x100 --output DP-0 --scale 1.5x1.5 --pos 3840x0" >> ~/.xprofile
  $ chmod +x ~/.xprofile
  ```

##### 6. Bash improvements

- customize `PS1` by setting `PS1="[\t] \u@\H:\w \$ "` in `~/.bashrc`
- install `bash-completion`
- install `pkgfile` and bootstrap his database by launching `pkgfile -u` as root
```
# pkgfile -u
```
- enable service `pkgfile-update.timer` to keep database updated
- add `source /usr/share/doc/pkgfile/command-not-found.bash` to `~/.bashrc`

##### 7. Git 

- install `git`
- add `source /usr/share/git/completion/git-prompt.sh` to `~/.bashrc`
- install `bash-git-prompt` and create symlink to installed scripts:
```
$ ln -s /usr/lib/bash-git-prompt ~/.bash-git-prompt
```
- pick a theme from the output of command `git_prompt_list_themes` and enable it by adding to `~/.bashrc`:
```
if [ -f "$HOME/.bash-git-prompt/gitprompt.sh" ]; then
    GIT_PROMPT_ONLY_IN_REPO=1
    GIT_PROMPT_SHOW_UPSTREAM=1
    GIT_PROMPT_THEME=Custom
    source $HOME/.bash-git-prompt/gitprompt.sh
fi
```
- to use a custom theme, do generate the `~/.git-prompt-colors.sh` file by doing the command `git_prompt_make_custom_theme`, and then customize it

##### 8. Pacman

- uncomment `Color` in `/etc/pacman.conf`
- increase the value of `ParallelDownloads` in `/etc/pacman.conf`

##### 9. Docker

- install `docker`, `docker-compose`
- add your user `username` to the `docker` group by doing `gpasswd -a username docker`
- enable `docker.service`

##### 10. Automatic backup with `rsync` (<https://wiki.archlinux.org/index.php/Rsync>) and timers (<https://wiki.archlinux.org/index.php/Systemd/Timers>)

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
- create the service `/etc/systemd/system/data-backup.service`:
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
- setup a timer by creating `/etc/systemd/system/data-backup.timer`:
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

##### 11. Gists

- [.bashrc for customized prompt with time and colored git](https://gist.github.com/disaverio/dd6929cb1ae5873dcf5675ee83311451) (as alternative to the `bash-git-prompt` previously described)
- [cutomized behaviour for Alt+Tab scoped to current workspace](https://gist.github.com/disaverio/4e53806a736764bcd571fca7643e4c34)
