# Pre-Installation

## Installation image and medium setup
Put USB flashdrive with archlinux-2021.10.01-x86_64.iso

VMWare Workstation -> new VM -> Typical -> Installer disc image file (iso): archlinux-2021.10.01-x86_64.iso -> Linux OS -> Other Linux 5.x and later kernel 64-bit

VM Name: ArchLinux

Default configurations except 2GB Ram and 20GB HDD

ArchLinux -> Open VM Directory -> Edit ArchLinux.vmx

Added firmware="efi" to ArchLinux.vmx file

## Boot the live environment and verify boot mode

Power on -> Arch Linux install medium (x86_64, UEFI)

Used # ls /sys/firmware/efi/efivars and confirmed it booted in UEFI mode since the command ran with no errors

## Connect to the internet

Used # ip link and confirmed ens33  (eth0) showed up -> rfkill list: not blocked

Did not need to set up wireless with iwctl

DHCP -> systemctl start dhcpcd@ens33.service -> systemctl enable dhcpcd@ens33.service -> ping archlinux.org: working

## Update the system clock

Used timedatectl set-ntp true -> checked timedatectl status -> checked timedatectl list-timezones -> entered timedatectl set-timezone America/Chicago

## Partition the disks

fdisk -l -> fdisk /dev/sda -> p -> Disklabel type: dos -> MBR

sgdisk -g /dev/sda -> convert MBR to GPT

No partitions -> fdisk /dev/sda: create new using n command -> default settings -> +512M -> t -> new partition -> change type to EFI System (code 1) -> n -> defaults -> w

## Format the partitions

Format partition -> # mkfs.fat -F32 /dev/sda1 -> mkfs.ext4 /dev/sda2

## Mount the partitions/filesystems

Mount partition -> mount /dev/sda2 /mnt

# Installation

## Select the mirrors

Select mirror -> reflector --verbose -c US -l 10 -p https -f 10 --save /etc/pacman.d/mirrorlist ->

## Install essential packages

pacstrap /mnt base linux linux-firmware vim nano

# Configure the System

## Fstab

Configure the system -> Generate fstab file: # genfstab -U /mnt >> /mnt/etc/fstab -> 

## Chroot

Change root into new system: # arch-chroot /mnt ->

## Timezone

Timezone: # ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime -> generate /etc/adjtime: # hwclock --systohc -->

## Localization

Localization: nano /etc/locale.gen -> uncomment en_US.UTF-8 UTF-8 -> save -> generate locales # locale-gen -> create locale.conf and set LANG variable: echo LANG=en_US.UTF-8 > /etc/locale.conf

## Network configuration

Network configuration -> create hostname file: echo archlinux > /etc/hostname -> add matching files to hosts: touch /etc/hosts -> 

127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname

systemctl enable dhcpd@ens33.service

systemctl enable systemd-networkd.service

## Set root password

Set root password -> passwd -> enter password

## Install bootloader

create directory for EFI partition: mkdir /efi -> mount /dev/sda1 /efi

Install bootloader -> pacman -S grub -> pacman -S efibootmgr -> 

create directory for EFI partition: mkdir /efi -> mount /dev/sda1 /efi

install GRUB: grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB-> grub-mkconfig -o /boot/grub/grub.cfg

# Reboot

Exit chroot: exit -> reboot

# Post-installation

Login in to root ->

## Install GUI

### Display server

Install GNOME desktop environment: xorg: pacman -S xorg-server

### Desktop Environment

-> pacman -S gnome -> systemctl start gdm.service

## System administration

Log back in as root -> change display settings to 2560 x 1600

add sudo --> open terminal -> pacman -Syu sudo
### Users and groups

### User management
Create a user account for yourself, sal, and codi with sudo permissions. The user names shall be "sal" and "codi" and the password shall be "GraceHopper1906" and be set to be changed after login.

Create user accounts: useradd -m -G wheel -s login_shell lily -> useradd -m -G wheel -s login_shell sal -> useradd -m -G wheel -s login_shell codi

Add sudo permissions: export EDITOR=nano -> visudo -> uncomment  %wheel   ALL=(ALL)   ALL

Change passwords: passwd lily -> GraceHopper1906 -> passwd sal -> GraceHopper1906 passwd codi -> GraceHopper1906

Require passwords to be changed after login: mark passwords as expired: chage -d 0 lily -> chage -d 0 sal -> chage -d 0 codi

## Console improvements

### Alternative shell
 Install a different shell other than bash, such as zsh or fish.
 
 pacman -S zsh
 
 Log out of root -> log into user account -> change password
 
 Go to terminal -> zsh -> go through zsh-newuser-install -> recommended default settings -> save
 
 Change default shell to zsh -> chsh -s /usr/bin/zsh -> log out and back in
 
 ### Install SSH
 
 open terminal -> sudo pacman -S openssh
 
 ssh into gateway -> ssh -p 53997 lil0722@129.244.245.21 -> continue connecting -> enter password -> connected to cognizant
 
 ### Add color coding
 
 sudo pacman -S zsh-syntax-highlighting -> sudo pacman -S zsh-autosuggestions -> sudo nano ~/.zshrc -> add 
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh

-> source ~/.zshrc
 
 ## Set the system to boot into the GUI desktop environment.
 
  -> systemctl enable gdm.service
  
 ## Add a few aliases to .bashrc or .zshrc.
 
 alias ls='ls --color=auto'
alias c='clear'
alias grep='grep --color=auto'
alias ping='ping -c 5'


-------------------------
## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/Lu-Lily/Lu-Lily.github.io/edit/main/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Lu-Lily/Lu-Lily.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
