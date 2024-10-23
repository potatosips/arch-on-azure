# arch-on-azure
timedatectl set-ntp true
cfdisk /dev/sda
mkfs.ext4 /dev/sda2
mount /dev/sda2 /mnt
pacstrap /mnt base linux linux-firmware grub nano git btop curl wget openssh dhcpcd sudo dnsutils base-devel htop
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Asia/Dhaka /etc/localtime
hwclock --systohc
nano /etc/locale.gen en_US.UTF-8
locale-gen
echo KEYMAP=us > /etc/vconsole.conf
echo "LANG=en_US.UTF-8" > /etc/locale.conf
grub-install --target=i386-pc /dev/sda
nano -w /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg
passwd
useradd -m -G wheel -s /bin/bash potato
passwd potato
nano /etc/sudoers
nano /etc/ssh/sshd_config
systemctl enable dhcpcd
systemctl enable sshd
nano -w /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 AzureVmTemplate AzureVmTemplate.localdomain
exit
umount -a
poweroff


nano -w /etc/mkinitcpio.conf #MODULES=(hv_storvsc hv_vmbus)
mkinitcpio -p linux
su (non root user)
mkdir aur
cd aur
git clone https://aur.archlinux.org/walinuxagent.git
cd walinuxagent
makepkg -si
systemctl enable waagent
nano /etc/waagent.conf
#ResourceDisk.EnableSwap=y
#ResourceDisk.SwapSizeMB=8192
	
mkdir  /mnt/resource
rm -Rf ~guytp/.bash_history ~guytp/aur ~/.bash_history
waagent -force -deprovision
export HISTSIZE=0
shutdown -h now

az login
az storage blob upload --account-name storage account name --account-key "access key from azure container" --container-name name of the container insite storage account  --name arch.vhd --file vhd location
