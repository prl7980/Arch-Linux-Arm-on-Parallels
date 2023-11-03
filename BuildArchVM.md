## Notes

* Prepare the build on an existing Linux VM, tested on Debian 12 and another Arch linux built with this procedure
* Do all work as su user
* Assumption of a 64GB disk, size accordingly
* A boot, root and swap partiton are created
* Assumption that the disk being worked on is /dev/sdb, change accordingly
* Build VM needs wget

## Create the disk partitions

cfdisk /dev/sdb

1 &emsp; efi  &emsp; &emsp;1GB  
2 &emsp; ext2 &emsp; 62GB  
2 &emsp; swap &emsp;1GB

## Format partitions

mkfs.fat -F32 /dev/sdb1      
mkfs.ext4 /dev/sdb2  

## Mount to the root filesystem

mkdir /mnt/arch  
mount /dev/sdb2 /mnt/arch  

## Create swap partition

mkswap /dev/sdb3  

## Deploy files to new disk

mkdir ~/arch  
cd ~/arch  
wget http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz  
bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C /mnt/arch  

## Chroot to new disk

mount --bind /mnt/arch /mnt/arch && cd /mnt/arch && rm /mnt/arch/etc/resolv.conf && cp /etc/resolv.conf etc && mount -t proc /proc proc && mount --make-rslave --rbind /sys sys && mount --make-rslave --rbind /dev dev && mount --make-rslave --rbind /run run

chroot /mnt/arch /bin/bash

## Pacman keys, settings and update

pacman-key --init  
pacman-key --populate archlinuxarm  
sed -i 's/#ParallelDownloads = 5/ParallelDownloads = 5/g' /etc/pacman.conf  
pacman -Syu vim  

## Install packages

pacman -Syu base linux linux-firmware vim arch-install-scripts efibootmgr networkmanager network-manager-applet dialog os-prober mtools dosfstools base-devel linux-headers net-tools inetutils

## Move boot files to boot partition
cp -r /boot/* /tmp/    
mount /dev/sdb1 /boot/    
cp -r /tmp/* /boot/     

## Generate machine id

dbus-uuidgen > /etc/machine-id

## Add fstab entries

blkid | grep /dev/sdb2 | sed 's/"//g' | awk '{print $5 "  /       ext4    defaults,noatime        0 1"}' >> /etc/fstab   
blkid | grep /dev/sdb1 | sed 's/"//g' | awk '{print $5 "  /boot   vfat    defaults                0 1"}' >> /etc/fstab   

## Timezones and locales
ln -sf /usr/share/zoneinfo/America/Vancouver /etc/localtime

hwclock --systohc

sed -i 's/#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen   
locale-gen

echo "LANG=en_US.UTF-8" >> /etc/locale.conf

## Hostname

echo "arch" > /etc/hostname   
echo -e "127.0.0.1\tlocalhost" >> /etc/hosts

## Install systemd boot manager

bootctl --path=/boot install

## Configure systemd boot manager

***Configure /boot/loader/loader.conf***

sed -i 's/#timeout 3/timeout 10/g' /boot/loader/loader.conf   
echo default arch.conf >> /boot/loader/loader.conf 

***Configure /boot/loader/entries/arch.conf***

echo -e "title    Arch Linux\nlinux       /Image\ninitrd  /initramfs-linux.img" > /boot/loader/entries/arch.conf  
blkid | grep /dev/sdb2 | sed 's/"//g' | awk '{print "options root="$5" rootfstype=ext4 rw rootflags=rw,noatime"}' >> /boot/loader/entries/arch.conf   

## Enable Network Manager

systemctl enable NetworkManager

## Enable ssh root user

sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

## Detach disk

* shutdown vm
* remove the build disk without deleting the file 

## Create VM with 

* Create a new VM














