Put USB flashdrive with archlinux-2021.10.01-x86_64.iso

VMWare Workstation -> new VM -> Typical -> Installer disc image file (iso): archlinux-2021.10.01-x86_64.iso -> Linux OS -> Other Linux 5.x and later kernel 64-bit

VM Name: ArchLinux

Default configurations except 2GB Ram and 20GB HDD

ArchLinux -> Open VM Directory -> Edit ArchLinux.vmx

Added firmware="efi" to ArchLinux.vmx file

Power on -> Arch Linux install medium (x86_64, UEFI)

Used # ls /sys/firmware/efi/efivars and confirmed it booted in UEFI mode since the command ran with no errors

Used # ip link and confirmed ens33 showed up

Did not need to set up wireless with iwctl

Used timedatectl set-ntp true -> checked timedatectl status -> checked timedatectl list-timezones -> entered timedatectl set-timezone America/Chicago

# fdisk -l -> fdisk /dev/sda -> p -> Disklabel type: dos -> MBR

# sgdisk -g /dev/sda -> convert MBR to GPT

No partitions -> create new using n command -> default settings -> +512M -> t -> new partition -> change type to EF -> n -> defaults -> w

Format partition -> # mkfs.fat -F32 /dev/sda1 -> mkfs.ext4 /dev/sda2

Mount partition -> mount /dev/sda2 /mnt

Select mirror -> pacman -Syy -> pacman -S reflector -> cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak -> reflector --verbose --country 'United States' -l 10 -p http --sort rate --save /etc/pacman.d/mirrorlist -> pacstrap /mnt base linux linux-firmware vim nano

Configure the system -> Generate fstab file: # genfstab -U /mnt >> /mnt/etc/fstab -> Change root into new system: # arch-chroot /mnt -> Timezone: # ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime -> generate /etc/adjtime: # hwclock --systohc --> Localization: nano /etc/locale.gen -> uncomment en_US.UTF-8 UTF-8 -> save -> generate locales # locale-gen -> create locale.conf and set LANG variable: echo LANG=en_US.UTF-8 > /etc/locale.conf -> export LANG=en_US.UTF-8

Network configuration -> create hostname file: echo archlinux > /etc/hostname -> add matching files to hosts: touch /etc/hosts -> 

127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname

Set root password -> passwd -> enter password

Install bootloader -> create directory for EFI partition: mkdir /efi -> mount /dev/sda1 /efi -> install GRUB: grub-install --taget=x86_64-efi --bootloader-id=GRUB --efi-directory=/efi -> # grub-mkconfig -o /boot/grub/grub.cfg

Exit chroot: exit -> reboot

Login in to root ->
Install GNOME desktop environment: xorg: pacman -S xorg -> pacman -S gnome



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
