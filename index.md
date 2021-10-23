# Pre-Installation

## Installation image and medium setup

To begin, prepare a USB flashdrive with `archlinux-2021.10.01-x86_64.iso`. Next, open VMWare Workstation and create a new VM with typical settings, Choose the installer disc image file (iso): `archlinux-2021.10.01-x86_64.iso`, Linux OS, and Other Linux 5.x and later kernel 64-bit. Name the VM "ArchLinux". Continue the setup with default configurations except 2GB of RAM and 20GB HDD. Next, right click on the ArchLinux VM and open the VM directory, then edit the `ArchLinux.vmx file`. Add the following line to the end of the file:
```
firmware="efi"
```

## Boot the live environment and verify boot mode

Power on the VM and hit "Enter" on Arch Linux install medium (x86_64, UEFI). Once the terminal prompt comes up, enter the following command.

```
# ls /sys/firmware/efi/efivars
```

If the command runs with no errors, the VM successfully booted in UEFI mode.

## Connect to the internet

First ensure that the network interface is listed and enabled:

```
# ip link
```

The results should show ens33 for ethernet. Next, configure the network connection with DHCP using the following commands.

```
# systemctl start dhcpcd@ens33.service
# systemctl enable dhcpcd@ens33.service
```
Test that the connection is working:

```
# ping archlinux.org
```

## Update the system clock

Ensure the system clock is accurate:

```
# timedatectl set-ntp true
# timedatectl status
```

Set the time zone:

```
# timedatectl set-timezone America/Chicago
```

## Partition the disks

Next, we need to partition disks for booting in UEFI mode (EFI system partition), and a partition for root. To begin, we will identify the block devices on the system.

```
# fdisk -l
```

The command shows /dev/sda exists as a block device. Now try

```
# fdisk /dev/sda
```

Now enter the `p` command to see that the "Disklabel type" is listed as dos, meaning the disk's partition table is MBR (Master Boot Record). We can convert the partition table to GPT (GUID Partition Table) using the following command:

```
# sgdisk -g /dev/sda
```

Now we will create the partitions. Enter

```
# fdisk /dev/sda
```

Then create a new partition using the `n` command. Follow the default settings, then set the last sector to +512M. Now use the `t` command and select the new partition. Change the partition type to EFI System (code 1). Now for the boot drive, enter `n` again and follow all default settings. Finally, enter the `n` command to write and exit `fdisk`.

## Format the partitions

To format partition the EFI system partition, enter the following command:

```
# mkfs.fat -F32 /dev/sda1
```

To format the root partition, enter the following command:

```
# mkfs.ext4 /dev/sda2
```

## Mount the partitions/filesystems

Mount the root partition:

```
# mount /dev/sda2 /mnt
```

# Installation

## Select the mirrors

To select suitable mirrors, we will use `reflector` like so:

```
# reflector --verbose -c US -l 10 -p https -f 10 --save /etc/pacman.d/mirrorlist
```

This will show a verbose listing of the fastest 10 most recently synchronized https mirrors and overwrites the mirrorlist.

## Install essential packages

To install the essential packages, run:

```
# pacstrap /mnt base linux linux-firmware vim nano
```

Any other packages can be appended onto the end if needed.

# Configure the System

## Fstab

Generate fstab file: 

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

## Chroot

Change root into the new system:

```
# arch-chroot /mnt
```

## Time zone

Set the time zone:

```
# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

Now generate `/etc/adjtime`: 

```
# hwclock --systohc
```

## Localization

Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8`. Generate the locales with:

```
# locale-gen
```

Create the `locale.conf` file and set the `LANG` variable:

```
# echo LANG=en_US.UTF-8 > /etc/locale.conf
```

## Network configuration

Create the hostname file: 

```
# echo archlinux > /etc/hostname
```
Configure the hosts file and add the following:

```
127.0.0.1        localhost
::1              localhost
127.0.1.1        archlinux
```

Enable DHCP:

```
# systemctl enable dhcpd@ens33.service
```

Enable the network manager:

```
# systemctl enable systemd-networkd.service
```

## Set root password

Set root password:

```
# passwd
```

Now enter any password desired.

## Install boot loader

Install the boot loader (GRUB) and `efibootmgr`:

```
# pacman -S grub
# pacman -S efibootmgr
```

Next, create a directory for the EFI partition and mount it:

```
# mkdir /efi
# mount /dev/sda1 /efi
```

Finally, execute the following commands to install GRUB and generate the configuration file:

```
# grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
```

# Reboot

Exit chroot with `exit` and restart the machine with `reboot`.

# Post-installation

Now we need to login into the new system with the root account.

## Install GUI

### Display server

We need a display server in order to setup the GUI:

```
# pacman -S xorg-server
```

### Desktop Environment

Now we will install GNOME as our desktop environment:

```
# pacman -S gnome
```

To start our new desktop environment, execute:

```
# systemctl start gdm.service
```

## System administration

Log back in as root.

### User management

We want to add sudo permissions. Open the terminal and install sudo:

```
# pacman -Syu sudo
```

Next I will create a user account for myself, sal, and codi with sudo permissions. First we begin by creating the user accounts:

```
# useradd -m -G wheel -s /bin/bash lily
# useradd -m -G wheel -s /bin/bash sal
# useradd -m -G wheel -s /bin/bash codi
```

This creates the accounts `lily`, `sal`, and `codi` as members of the `wheel` group. We will add sudo permissions to the accounts with the following steps. We will set our default editor to nano:

```
# export EDITOR=nano
```

Next we will edit `/etc/sudoers` using

```
# visudo
```

We will then uncomment `%wheel   ALL=(ALL)   ALL` and save it, thus granting all members of the `wheel` group sudo permissions.

To set the passwords of the accounts to `GraceHopper1906` we will run

```
# passwd lily
# passwd sal
# passwd codi
```

one by one and set the passwords as desired. To require the passwords to be changed after login, we will mark the passwords as expired:

```
#chage -d 0 lily
# chage -d 0 sal
# chage -d 0 codi
```

## Console improvements

### Alternative shell

We will install zsh as an alternative shell:

```
# pacman -S zsh
```

Now we will log out of root and into our user account where we will be prompted to change the password. After doing so, we will go to the terminal, enter

```
# zsh
```

and then go through `zsh-newuser-install`. We will keep to the recommended default settings and save. To change the default shell to zsh, we run:

```
# chsh -s /usr/bin/zsh
```

Then we will log out and log back in.
 
### Install SSH

To install SSH, we will open the terminal and execute the following command:

```
# sudo pacman -S openssh
```

Once installed, we SSH into the gateway with:

```
# ssh -p 53997 lil0722@129.244.245.21
```
 
### Add color coding

To add syntax highlighting to zsh, we execute the following:

```
# sudo pacman -S zsh-syntax-highlighting
# sudo pacman -S zsh-autosuggestions
# sudo nano ~/.zshrc
```

Next we add the following to `~/.zshrc`:

```
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```

Next, source zsh with:

```
# source ~/.zshrc
```
 
## Set the system to boot into the GUI desktop environment.

To set the system to boot into the GUI desktop enviroment, we execute this command:

```
# systemctl enable gdm.service
```
  
## Add a few aliases to .bashrc or .zshrc.

We use the `alias` command to add aliases. These are some aliases for zsh:

```
# alias ls='ls --color=auto'
# alias c='clear'
# alias grep='grep --color=auto'
# alias ping='ping -c 5'
```
