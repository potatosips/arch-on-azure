# Arch Linux Virtual Machine Setup for Azure (BIOS Mode)

This guide outlines the steps to create an Arch Linux VM on Oracle VirtualBox, configure it for Azure, and upload the VHD to an Azure storage account.

## Prerequisites

- Oracle VirtualBox (BIOS mode)
- A 30GB fixed-size VHD disk
- Bridge networking enabled for SSH access

---

## Arch Linux Installation

**Start the VM after making the disk:**


timedatectl set-ntp true
cfdisk /dev/sda


Create a 10MB partition with the BIOS boot type.
Create the remaining space with the Linux filesystem type.
Format and mount the filesystem:


mkfs.ext4 /dev/sda2   # Adjust partition number if needed (e.g., sda2 for 30GB).
mount /dev/sda2 /mnt


Install the base system and utilities:
pacstrap /mnt base linux linux-firmware grub nano git btop curl wget openssh dhcpcd sudo dnsutils base-devel htop


Generate the fstab:
genfstab -U /mnt >> /mnt/etc/fstab


Chroot into the system:
arch-chroot /mnt


Set timezone and hardware clock:
ln -sf /usr/share/zoneinfo/Asia/Dhaka /etc/localtime
hwclock --systohc

Configure locale:
nano /etc/locale.gen  # Uncomment en_US.UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

Set up keyboard layout:
echo "KEYMAP=us" > /etc/vconsole.conf

Install GRUB bootloader:
grub-install --target=i386-pc /dev/sda
nano /etc/default/grub  # Edit if needed, then save and exit.
grub-mkconfig -o /boot/grub/grub.cfg

Set root password:
passwd

Create a user and set password:
useradd -m -G wheel -s /bin/bash potato  # Change 'potato' to your username if desired.
passwd potato

Edit sudoers file:
nano /etc/sudoers  # Uncomment %wheel and %wheel ALL=(ALL) NOPASSWD: ALL

Configure SSH:
nano /etc/ssh/sshd_config  # Uncomment PermitRootLogin and set to yes

Enable services:
systemctl enable dhcpcd
systemctl enable sshd

Set up /etc/hosts:
nano /etc/hosts
Add the following lines:
127.0.0.1    localhost
::1          localhost
127.0.1.1    AzureVmTemplate AzureVmTemplate.localdomain

Exit chroot and unmount:
exit
umount -a

Power off the VM:
poweroff

Prepare for Azure
After booting, modify mkinitcpio.conf:
nano /etc/mkinitcpio.conf
Change the line MODULES=() to:
MODULES=(hv_storvsc hv_vmbus)

Rebuild initramfs:
mkinitcpio -p linux

Switch to your user and prepare for AUR:
su potato
cd
mkdir aur
cd aur
git clone https://aur.archlinux.org/walinuxagent.git

Build and install walinuxagent:
cd walinuxagent
makepkg -si

Enable waagent:
systemctl enable waagent

Configure waagent:
nano /etc/waagent.conf

Change the following lines:
ResourceDisk.EnableSwap=y
ResourceDisk.SwapSizeMB=8192

Create resource disk mount point:
mkdir /mnt/resource

Clean up:
rm -Rf ~guytp/.bash_history ~guytp/aur ~/.bash_history

Deprovision the VM for Azure:
waagent -force -deprovision
export HISTSIZE=0
shutdown -h now

Install Azure CLI following this guide and then run
guide https://learn.microsoft.com/en-us/cli/azure/install-azure-cli
az login (complete login)
az storage blob upload --account-name <storage account name> --account-key <access key from azure container> --container-name <name of the container insite storage account>  --name arch.vhd --file <vhd location>
