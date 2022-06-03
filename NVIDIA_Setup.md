# [NVIDIA](https://wiki.archlinux.org/title/NVIDIA) setup

Instructions to setup NVIDIA with drm in kms, and Wayland up & running.

The instructions are based on hardware with NVIDIA dedicated graphic card and integrated Intel graphic card. In particular:
- motherboard gryphon z97 with integrated hd graphic card (HD Graphics 4600, via i7 4790k)
- nvidia graphic card: Asus GeForce GTX 1050TI

```
$ lspci -k | grep -A 3 -E "(VGA|3D)"
02:00.0 VGA compatible controller: NVIDIA Corporation GP107 [GeForce GTX 1050 Ti] (rev a1)
	Subsystem: ASUSTeK Computer Inc. Device 862a
	Kernel driver in use: nvidia
	Kernel modules: nouveau, nvidia_drm, nvidia
```

**1.** Install needed packages: `nvidia`, `nvidia-utils`, `egl-wayland`

**2.** Enable DRM [KMS](https://wiki.archlinux.org/title/Kernel_mode_setting), by adding `nvidia-drm.modeset=1` [kernel parameter](https://wiki.archlinux.org/title/Kernel_parameters):

- edit `/etc/default/grub` and add the `nvidia-drm.modeset=1` option between the quotes in the `GRUB_CMDLINE_LINUX_DEFAULT` line. Result:
```
$ cat /etc/default/grub | grep GRUB_CMDLINE_LINUX_DEFAULT
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1"
```

- re-generate the `grub.cfg`:
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

- result:
```
$ cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-linux root=UUID=... rw loglevel=3 quiet nvidia-drm.modeset=1
```

**3.** To early start KMS (assuming you are using `mkinitcpio`):

- add  `nvidia`, `nvidia_modeset`, `nvidia_uvm`, `nvidia_drm` modules to `MODULES=( ... )` array in `/etc/mkinitcpio.conf` ([details](https://wiki.archlinux.org/title/Mkinitcpio#MODULES))

- regenerate presets by running:
```
# mkinitcpio -P
```

- to avoid forgetting to update initramfs after an NVIDIA driver upgrade, do setup a pacman hook by creating the file `/etc/pacman.d/hooks/nvidia.hook`:
```
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux

[Action]
Description=Update Nvidia module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

**4.** Set framebuffer resolution by editing following vairables in `/etc/default/grub`:
```
GRUB_TERMINAL_OUTPUT="gfxterm"
GRUB_GFXMODE=1920x1080,auto
GRUB_GFXPAYLOAD_LINUX=keep
```

- re-generate the `grub.cfg`:
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

**5.** Comment the `WaylandEnable=false` line in the `/etc/gdm/custom.conf`. Result:
```
$ cat /etc/gdm/custom.conf | grep Wayland
#WaylandEnable=false
```

**6.** Install `xorg-xwayland`
