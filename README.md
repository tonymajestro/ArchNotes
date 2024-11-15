# Base install
## Initialize ssh and change root password
passwd (change root password)
systemctl enable sshd

## Check internet 
ping archlinux.org

## Update system clock
timedatectl

## Partition drive
lsblk (find the drive)
fdisk /dev/sda
+1G for boot (change type with uefi alias)
+40G for swap (change type with swap alias)
rest for root

## Format partitions
mkfs.ext4 /dev/sda3
mkswap /dev/sda2
mkfs.fat -F 32 /dev/sda1

## Mount filesystems
mount /dev/sda3 /mnt
mount --mkdir /dev/sda1 /mnt/boot
swapon /dev/sda2

## Install packages
### Base
packstrap -K /mnt base linux linux-firmware

### Microcode updates
pacstrap -K /mnt intel-ucode
vim /etc/mkinitcpio.conf (add microcode to HOOKS array, after autodetect)

### Other packages 
pacstrap -K /mnt neovim man-db man-pages texinfo networkmanager grub efibootmgr sudo

## Configure fstab
genfstab -U /mnt >> /mnt/etc/fstab

## Date/time/locale (in chroot)
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime
hwclock --systohc

vim /etc/locale.gen (uncomment en_US.UTF-8 UTF-8)
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

## Networking (in chroot)
echo 'hostname' > /etc/hostname
systemctl enable NetworkManager

## Initialize ramdisk (in chroot)
mkinitcpio -P

## Set root password (in chroot)
passwd (change root password)

## Grub (in chroot)
grub-install --target=x86_64-efi --efi-directory /boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

## Reboot
exit (exit chroot)
unmount -R /mnt
reboot

## Test installation
log in as root
ping archlinux.org

## Random issues
```
neovim: signature from "Daniel M. Capella <polyzen@archlinux.org>" is unknown trust"
``````
Solution: update pacman archlinux-keyring
- pacman -S archlinux-keyring
