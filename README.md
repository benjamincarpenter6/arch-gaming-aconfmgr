# arch-gaming-aconfmgr
Ben’s PURE GAMING Arch Guide

Verify internet connection
ping gnu.org

Update the system clock
timedatectl

Partition the disks 
sgdisk -Z /dev/the_disk_to_be_partitioned

sgdisk -n1:0:+1G -t1:ef00 -N2 -t2:8304 /dev/the_disk_to_be_partitioned

mkfs.ext4 /dev/root_partition

mkfs.fat -F 32 /dev/efi_system_partition

Mount the file systems
mount /dev/root_partition /mnt

mount --mkdir /dev/efi_system_partition /mnt/efi

Select the mirrors
reflector --save /etc/pacman.d/mirrorlist

Install essential packages
pacstrap -K /mnt base

Chroot
arch-chroot /mnt

Time
ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime

hwclock --systohc

systemctl enable systemd-timesyncd.service

Localization
sed -i '/^#.*en_US.UTF-8 UTF-8/s/^#//' /etc/locale.gen

locale-gen

echo 'LANG=en_US.UTF-8' > /etc/locale.conf

echo 'KEYMAP=us' > /etc/vconsole.conf

Network configuration
echo hostname >> /etc/hostname

pacman -S networkmanager wireless-regdb

*uncomment US /etc/conf.d/wireless-regdom

systemctl enable NetworkManager.service

Root password
passwd

Boot loader
bootctl install

systemctl enable systemd-boot-update.service

echo 'default @saved
timeout 0
console-mode auto' > /efi/loader/loader.conf

Swap file
mkswap -U clear --size 4G --file /swapfile
swapon /swapfile

echo '[Swap]
What=/swapfile

[Install]
WantedBy=swap.target' >> /etc/systemd/system/swapfile.swap

systemctl enable swapfile.swap

echo "vm.swappiness = 10" >> /etc/sysctl.d/99-swappiness.conf

CachyOS optimized repositories (check if it’s znver)
curl https://mirror.cachyos.org/cachyos-repo.tar.xz -o cachyos-repo.tar.xz

tar xvf cachyos-repo.tar.xz && cd cachyos-repo

./cachyos-repo.sh

Chaotic-AUR repository
pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com

pacman-key --lsign-key 3056513887B78AEB

pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'

pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'

echo '[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist' >> /etc/pacman.conf

Multilib repository
awk '/^\s*#(\[multilib\]|Include\s*=\s*\/etc\/pacman\.d\/mirrorlist)/ { sub(/^\s*#/, ""); } { print }' /etc/pacman.conf


pacman -Syu linux-cachyos linux-cachyos-headers linux-cachyos-lts linux-cachyos-lts-headers chwd amd-ucode dosfstools e2fsprogs linux-firmware sof-firmware networkmanager vim man-db man-pages tldr sudo reflector

Fonts
pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra

Mkinitcpio UKI
echo 'HOOKS=(systemd autodetect microcode modconf kms sd-vconsole block filesystems fsck)' > /etc/mkinitcpio.conf.d/custom.conf

# mkinitcpio preset file for the 'kernel' package

#ALL_config="/etc/mkinitcpio.conf"
ALL_kver="/boot/vmlinuz-kernel"

PRESETS=('default')

#default_config="/etc/mkinitcpio.conf"
#default_image="/boot/initramfs-kernel.img"
default_uki="/efi/EFI/Linux/arch-kernel.efi"
#default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"

#fallback_config="/etc/mkinitcpio.conf"
#fallback_image="/boot/initramfs-kernel-fallback.img"
#fallback_uki="/efi/EFI/Linux/arch-kernel-fallback.efi"
#fallback_options="-S autodetect"

mkinitcpio -P

Hardware detection
chwd -a

Reboot
exit

umount -R /mnt

reboot

Post-install

Create a user
useradd -m -G wheel -s login_shell username

passwd username

echo '%wheel ALL=(ALL:ALL) ALL' > /etc/sudoers.d/01_allow_wheel

Security
pacman -S ufw

ufw default deny incoming

ufw default allow outgoing

systemctl enable ufw

Service management
systemctl enable --now reflector.timer
systemctl enable --now fstrim.timer

KDE Plasma
pacman -S plasma-login-manager plasma-desktop

Misc tweaks


~/.config/environment.d/
PROTON_FSR4_UPGRADE=1
PROTON_ENABLE_WAYLAND=1
PROTON_ENABLE_HDR=1
PROTON_USE_NTSYNC=1
PROTON_LOCAL_SHADER_CACHE=1
ENABLE_LAYER_MESA_ANTI_LAG=1
MESA_SHADER_CACHE_MAX_SIZE=12G


Cachyos settings
systemctl mask power-profiles-daemon
echo 'rootflags=noatime systemd.zram=0 nowatchdog' >> /etc/kernel/cmdline 

Uncomment multilib


Ext4 performance

Ssd power management


Pipewire setup
pacman -S pipewire wireplumber











nvme_core.default_ps_max_latency_us=0 to completely disable APST, or set a custom threshold to disable specific states.
If setting latency still does not works, try adding pcie_aspm=off and pcie_port_pm=off

Util-linux
