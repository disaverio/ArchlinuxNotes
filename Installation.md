# Archlinux installation for UEFI systems and GPT


## Before starting

**1.** Download .iso and .sig from <https://www.archlinux.org/download/>

Verify signature:
```
$ gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```
Prepare USB installation media as UEFI bootable. **Replace `/dev/SDX` with correct value**:
```
# dd bs=4M if=path/to/archlinux.iso of=/dev/SDX status=progress oflag=sync
```

**2.** Eventually set the AHCI mode and turn of Secure Boot.

**3.** Eventually prepare the disk by reserving space for the installation (if the installation has to be done in an already existing Windows setup, otherwise ignore this point).

## Prepare the system

**1.** Boot system from prepared media and **select UEFI boot**

**2.** Set keyboard layout:
```
# loadkeys it
```

##### 3. Connect to internet via WiFi (<https://wiki.archlinux.org/index.php/Network_configuration/Wireless>)

- check if your wifi card is recognized, it should be listed by:
```
# ip link
```
- enable iwd service:
```
# systemctl start iwd.service
```
- run it in interactive mode:
```
# iwctl
```
- then list again your devices to get your wifi device name (e.g. `wlan0`), scan for networks, and list them:
```
[iwd]# device list
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
```
- finally connect to the network (you will be prompted for a passphrase):
```
[iwd]# station wlan0 connect SSID
```
- CTRL+D to leave `iwd`, and get addresses via dhcp:
```
# dhcpcd wlan0
```
- check it pinging something

**4.** Update system clock:
```
# timedatectl set-ntp true
```

##### 5. Partition the disk. (<https://wiki.archlinux.org/index.php/GPT_fdisk>)

- list all the disks to identify the correct device:
```
# fdisk -l
```
- run `gdisk` on such device, to enter interactive mode. **Replace `/dev/SDX` with correct value**:
```
# gdisk /dev/SDX
```

Now two possible scenarios are the most common use cases:

**5.1.** Wipe the whole disk: all the data will be destroyed, a new GUID partition table (GPT) will be created (removing an old MBT partition table if exists), and two partitions will be created: a 512MB EFI partition (mandatory) and a Linux one for the system:

- create a new empty GUID partition table by entering `o`
- create new EFI partition by entering `n`. *Notes*: default as first sector, +512M as last sector, **`ef00` as partition hex code**
- create new Linux partition for the system by entering `n` again. *Notes*: default as first sector, default as last sector (the whole disk), default as partition hex code (`8300`)
- save and exit entering `w`

**5.2.** Installation in an already existing Windows setup: no need for a EFI partition:

- create new Linux partition for the system by entering `n` again. *Notes*: default as first sector, default as last sector (the whole disk), default as partition hex code (`8300`)
- save and exit entering `w`

##### 6. Format the partitions

- format the EFI partition (let's say number 1). **Replace `/dev/SDX1` with correct value**:
```
# mkfs.fat -F32 /dev/SDX1
```
- format the Linux partition (let's say number 2) **Replace `/dev/SDX2` with correct value**:
```
# mkfs.ext4 /dev/SDX2
```

##### 7. Mount the partitions

- mount the system partition:
```
# mount /dev/sdX2 /mnt
```
- create `/boot` folder to mount the EFI partition:
```
# mkdir /mnt/boot
```
- mount EFI partition:
```
# mount /dev/sdX1 /mnt/boot
```
- if the EFI partition's size is not big enough, let's say less than 256MB, do not use the `boot` folder. Create a `efi` folder instead, and mount it there. In this way future kernel updates will not consume the little EFI partition' space:
```
# mkdir /mnt/efi
# mount /dev/sdX1 /mnt/efi
```

## Installation

**1.** Select mirrors: move the best entries on top of the list `/etc/pacman.d/mirrorlist` (most likely the best ones are the ones from your country)

**2.** Install base system along with network utilities, and text editor:
```
# pacstrap /mnt base linux linux-firmware iwd dhcpcd nano
```

**3.** Generate `fstab`:
```
# genfstab -U /mnt >> /mnt/etc/fstab
```

**4.** Change root into the new system:
```
# arch-chroot /mnt
```

**5.** Set time zone:
```
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
# hwclock --systohc
```

##### 6. Localization

- uncomment `en_US.UTF-8 UTF-8` (and eventually others) in `/etc/locale.gen` and then generate them with:
```
# locale-gen
```
- set the language and terminal keyboard layout:
```
# echo LANG=en_US.UTF-8 > /etc/locale.conf
# echo KEYMAP=it > /etc/vconsole.conf
```

**7.** Network. **Replace myhostname**:
```
# echo myhostname > /etc/hostname
# echo 127.0.0.1 localhost > /etc/hosts
# echo ::1 localhost >> /etc/hosts
# echo 127.0.0.1 myhostname.localdomain myhostname >> /etc/hosts
```

**8.** Set root password:
```
# passwd
```

##### 9. GRUB Boot loader (<https://wiki.archlinux.org/index.php/GRUB>)

- installation in the arch system with support to efi:
```
# pacman -S grub efibootmgr
```
- install grub in the disk. **Replace ESP with mount point of efi partition. In this guide `/boot` (or `/efi`, it depends on what you did on point #5)**:
```
# grub-install --target=x86_64-efi --efi-directory=ESP --bootloader-id=GRUB
```
- generate configuration file
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

##### 10. Enable Microcode to fix cpu architecture bug (<https://wiki.archlinux.org/index.php/Microcode>)

Enable if your cpu is affected (e.g.it is if Haswell, Broadwell among others): <https://en.wikipedia.org/wiki/Transactional_Synchronization_Extensions>

Install `intel-ucode` (or the `amd` version) and then update GRUB to early load microcode, before initramfs stage:
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

**11. FINISH**: CTRL+D to exit `chroot` env and `reboot`
