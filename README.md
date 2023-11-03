# Arch Install

## Downloaded ISO

Downloaded ISO from the download page on the wiki, via MIT.edu

Verified signature via Windows PowerShell using these commands/inputs in windows powershell

```
Get-FileHash
<PATH FOR ISO>
```

Compared resulting SHA256 checksum to the given checksum on the downloads page

## Setting up on VMWare

Created a new VM using the ISO
Linux version: Other Linux 5.x kernel 64 bit
Named VM ArchLinux, default location
Allocated 1 processor, 2 cores
Allocated 2 GB ram
Used default Network type (NAT), I/O Controller Type (LSI), Disk Type (SCSI), 
Create a new virtual disk with 20 GB of space, stored as a single file
Finish creation and boot VM

Once boot page is up, shut down machine
Open VM directory from VMware and edit the file Arch Linux.vmx, inserting as line 2 `firmware="efi"` to allow it to boot in EUFI mode

Next, I loaded the VM and booted on EUFI mode

## Set Keyboard

As default keyboard is US, skip to the next step

## Verify Boot Mode
If the command `cat /sys/firmware/efi/fw_platform_size` returns without errors, booted via EUFI correctly

## Connect to the internet

Verify internet connection with command `ip link`
optional: `ping archlinux.org` to double check that internet is working

## Update the system clock

run command `timedatectl` to ensure system clock accurate

## Partition the disks

run command `fdisk -l` to see available disks

run the following commands to partition the virtual disk, creating a partition of size 500 MiB and one of size 19.5 GB
```bash
fdisk /dev/sda
n # new partition
p # primary partition
<enter to assign first partition default value 1>
<enter to default start location>
+500M

n
p
<enter: default value 2>
<enter: default start>
<enter: default end>

w # write changes
```

checked partition table with `p` before writing

initially, I was throwing `unknown error 524` every time I tried to run `fdisk /dev/sda` but that resolved once I made a new vm with source files in my disk rather than my onedrive.

## Format Partitions
run `mkfs.fat -F 32 /dev/sda1` to format the boot partition
run `mkfs.ext4 /dev/sda2` to format the root partition

## Mount Filesystems
Mount the root volume to /mnt with `mount /dev/sda2 /mnt` 

Mount the EFI system partition with `mount --mkdir /dev/sda1 /mnt/boot`

can check mounts with `findmnt`

## Install essential packages
install the base package with `pacstrap -K /mnt base linux linux-firmware`

## Configure system

### Fstab
Generate an fstab file using `genfstab -U /mnt >> /mnt/etc/fstab`

### Change root into new system
```arch-chroot /mnt```

### Set time zone 
`ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`

run `hwclock --systohc` to generate /etc/adjtime

### Localization
use nano to edit /etc/locale.gen and uncomment `en_US.UTF-8 UTF-8`

Generate locales by running `locale-gen`

Create the locale.conf(5) file, and set the LANG variable with:
```
echo 'LANG=en_US.UTF-8' > /ect/locale.conf
```

### Network configuration
create the hostname file
```echo 'Mason' /etc/hostname```

### Root password
set the root password with `passwd`

### Boot loader
I chose GRUB and ran the following commands to download and configure it
```
pacman -Syu grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### Install Packages

after chrooting, you can install optional (but recommended) packages

nano (text editor)
```
pacman -S nano
```

network manager
```
pacman -S networkmanager
systemctl enable NetworkManager
```

Dhclient
```
pacman -S dhclient
```

Firewall
```
pacman -S ufw
systemctl enable ufw
```

ModemManager
```
 pacman -S modemmanager
 systemctl enable ModemManager
```
RAID and LVM Management Utilities:
```
pacman -S mdadm lvm2
```

info and man pages
```
pacman -S texinfo
pacman -S man-db
pacman -S man-pages
```
## Exit and Reboot
Exit using ctl+d or `exit`
Reboot with `reboot`

## Installing a DE
I chose XFCE because I like rats and the logo is a rat

```bash
pacman -Syu # update system

pacman -S xfce4 xfce4-goodies
# install a display manager (LightDM)
pacman -S lightdm
systemctl enable lightdm
systemctl start lightdm
```

### Troubleshooting LightDM

Press CTRL+ALT+F2 to reach console mode
```
systemctl status lightdm
```
If the status is GREEN, something else is broken

```
lightdm --testmode --debug
```
run this code to see where the errors are

In my case, it wasn't finding the greeter, so I took the following steps to debug

```bash
pacman -S lightdm-gtk-greeter
systemctl enable lightdm
systemctl reboot
```

Doing all this should boot the GUI by default

## Add New Users and set Passwords
```
useradd -m Codi
useradd -m Mason

passwd Codi # GraceHopper1906
passwd Mason 
```

Give sudo permissions
```bash
pacman -S sudo vi

visudo
<uncomment %sudo ALL=(ALL) ALL to give sudo users permissions>
<esc> :x # write changes

groupadd sudopop
usermod -aG sudopop Codi
usermod -aG sudopop Mason

visudo 
<add line %sudopop ALL=(ALL) ALL>
```

## Add a Shell
I chose fish because I like fish and I like the animal theme I have going on
For usability purposes, I did so at a later point

After installing fish using `sudo pacman -S fish` and entering the shell using `fish`, I decided I didn't like it that much and pivoted to zsh. It seems cool but a hassle at the moment. A hassle that I normally would tackle due to intruigue, but I should not now due to time constraints. A rare moment of self control. Not that rare. I should belive in myself. 

Bye-bye fish. `sudo pacman -R fish`

Hello zsh:
The following commands downloaded, configured, and set zsh as the default shell
```
sudo pacman -S zsh
zsh

<walkthrough of zsh config, setting up deafult parameters for the most part>

chsh -s /usr/bin/bash
```

### Customizing Zsh
```zsh
nano ~/.zshrc
 # add the following lines
PROMPT="%S%F{009}%n%f%s@%F{magenta}%m%f %F{green}%1~%f $ "
# colors! and more information in the prompt

alias c='clear'
alias ls='ls --color=auto'
alias ll='ls -la'
# adding aliases

<save and exit .zshrc>

source .zshrc # apply changes
```

## Download SSH
```bash
sudo pacman -S openssh
```

## Install a browser
```zsh
sudo pacman -Sy firefox
```
