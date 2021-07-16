# arch

## Pre-installing

- If you're testing it on "VirtualBox" remember to enable UEFI
    - > Settings > System > Motherboard > Enable EFI(special OSes only)

### Keyboard
1. Find the keymap
    - > ls /usr/share/kbd/keymaps/**/.map.gz | grep br
2. Setting keymap
    - > localectl set-keymap --no-convert br-abnt2

### Connect to internet

#### Wifi
1. Enter `iwctl` shell
    - > iwctl
2. Follow the steps with:
    - > help
3. Exit with:
    - > quit

### Update system clock
1. Just run
    - > timedatectl set-ntp true

### Partition disks

#### Preparing
1. Get disk list
    - > fdisk -l
    - ```
         Disk /dev/sda: 32GiB .......
         Disko model: ......
      ```
2. Select disk to partition
    - > fdisk /dev/sda

#### Layout
- I'll be following the `UEFI with GPT`
| Mount Point           | Partition                 | Partition Type        | Suggested Size          |
| --------------------- | ------------------------- | --------------------- | ----------------------- |
| /mnt/boot or /mnt/efi	| /dev/efi_system_partition	| EFI system partition	| 550M                    |
| [SWAP]	              | /dev/swap_partition	      | Linux swap	          | 2G                      |
| /mnt	                | /dev/root_partition	      | Linux x86-64 root (/)	| Remainder of the device |

##### EFI System Partition
1. Press `g`
    - Will create a Disk label
2. Press `n`
    - To add a new partition
3. Partition number
    - Default (just press enter)
4. First Sector
    - Default (press enter)
5. Last sector
    - > +550M

##### Swap Partition
1. Press `n`
    - To add a new partition
2. Partition number
    - Default (just press enter)
3. First Sector
    - Default (press enter)
4. Last sector
    - > +2G

##### Filesystem Partition
1. Press `n`
    - To add a new partition
2. Partition number
    - Default (just press enter)
3. First Sector
    - Default (press enter)
4. Last sector
    - Default (press enter)

##### Changing the partitions types
1. Press `t`
    - > Partition number: 1
    - > Partition type: 1 // EFI System
2. Press `t`
    - > Partition number: 2
    - > Partition type: 19 // Linux SWAP

##### Write tables and exit
1. Just prest `w`

#### Format the partitions
1. EFI System - FAT32
    - > mkfs.fat -F32 /dev/sda1
2. SWAP System - SWAP
    - > mkswap /dev/sda2 
    - Turn swap on
    - > swapon /dev/sda2 
3. Linux filesystem - Ext4
    - > mkfs.ext4 /dev/sda3 

#### Mount the file system
- > mount /dev/sda3 /mnt

## Installing
1. Using `pacstrap` to install `base` package, `linux`(Linux kernel) and `linux-firmware`(firmware for the hardware)
    - > pacstrap /mnt base linux linux-firmware

### Configure the system
1. Generate `fstab` file
    - > genfstab -U /mnt >> /mnt/etc/fstab
2. Change root into the new system
    - > arch-chroot /mnt
    - This will enter into arch shell

#### Set timezone 
1. Get timezone list
    - > ls /usr/share/zoneinfo/**/* | grep Sao_Paulo
2. Set timezone
    - > ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime

#### Adjust clock
> hwclock --systohc

#### Set locale
1. Generate locale
    - > locale-gen
2. Install a text editor
    - > pacman -S vim
3. Create a `local.conf`
    - > vim /etc/locale.conf
    - and add:
    - ```
          LANG=en_US.UTF8
      ```
    - (Alternative)
        - > localectl set-locale LANG=en_US.UTF8

#### Persistent keyboard layout
1. Create `vconsole.conf`
    - > vim /etc/vconsole.conf
    - and add:
    - ```
          KEYMAP=br-abnt2
      ```

#### Network configuration
1. Define `hostname`
    - > vim /etc/hostname
    - and add any name
    - ```
          archrafa
      ```
2. Create `/etc/hosts`
    - > vim /etc/hosts
    - and add any name
    - ```
          127.0.0.1     localhost
          ::1           localhost
          127.0.1.1     archrafa.localdomain archrafa
      ```

3. Add root password
    - > passwd
    - And type you password

#### Add user
1. > useradd -m rafa
2. Give it some permission
    - > usermod -aG wheel,audio,video,optical,storage rafa

#### Give sudo privileges
1. Install sudo
    - > pacman -S sudo
2. Give `whell` permission
    - > EDITOR=vim visudo
3. Look for the line: _# %wheel ALL=(ALL) ALL_
4. Uncomment it

#### Boot loader
1. Install some packages
    - > pacman -S efibootmgr dosfstools os-prober mtools grub
    - Here you can also install any package that you want: git, asdf, ...
    - But I want to use `yay` to manage those packages
2. Mount into `/boot/EFI`
    - > mkdir /boot/EFI
    - > mount /dev/sda1 /boot/EFI
3. Installing grub
    - > grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
4. Generating `grub.cfg`
    - > grub-mkconfig -o /boot/grub/grub.cfg

#### Installing some packages
- > pacman -S networkmanager
    - Just to ease the internet configuration
    > systemctl enable NetworkManager
- > pacman -S git

#### Rebooting
- > exit
- > umount -l /mnt
- If you on VirtualBox
    - > shutdown now
    - Remove the ISO, and start the VM
- If you are on you machine
    - > reboot
    - Remove the pendrive, or select another boot method on BIOS, idk

## Xorg
- A display server

### Installing
- > pacman -S xorg-server
- > pacman -S xorg xorg-xinit
- Driver Installation
    - > pacman -S xf86-video-intel
    - Because Im using Intel Graphics
- __important__ if you're using VirtualBox you need to instead install:
    - > `pacman -S xf86-video-vesa`
    - > `pacman -S virtualbox-guest-utils`


### Config files:
- Can be edited at: 
    - `/etc/X11/xorg.conf`
    - `/etc/xorg.conf`

## KDE Plasma
- Desktop Environment

### Installing
1. > pacman -S plama-meta
    - > 2 //noto-fonts
    - > 1 // phonon-qt5-gstreamer
    - > Y

## SDDM
- Display Manager
1. > pacman -S sddm

## i3-gaps
- Tiling window manager

### Installing
1. > pacman -S i3-gaps
2. > pacman -S i3status // optional

### Configuring
1. > vim ~/.config/.xinitrc
2. Comment these:
```
twm &
// and anything between them
exec xterm....
```
3. Add: `exec i3`

## Polybar
1. > pacman -S polybar

## Configuring
1. Enable sddm
    - > sudo systemctl enable sddm
2. Enable NetworkManager
    - > sudo systemctl enable NetworkManager
3. Add `sddm` config file
    - > sudo vim /usr/lib/sddm/sddm.conf.d/default.conf
4. Reboot
    - > sudo systemctl reboot
