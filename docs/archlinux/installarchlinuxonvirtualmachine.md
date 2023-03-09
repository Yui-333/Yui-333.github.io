# Install Archlinux On Virtual Machine

First set password on root account using
```
passwd
```

Then start ssh service
```
systemctl start sshd
```

Find IP address of the VM, you can use:
```
ip a
```

Once you have IP for the VM, go to a computer with SSH enabled terminal/console and login to the VM using ‘root’ as username and password you have setup previously, example:
```
ssh root@[ip address]
```

For the VM I am using a 30GB vDisk, which will be partitioned in following way:
```
/boot - 512MB
[SWAP] - 2GB
/ - ~remaining
```

Check Disk
```
fdisk -l
```
In my case, this is ‘sda’.

Start fdisk to format the disk.
```
fdisk /dev/sda
```

Format partitions to a specific file system:

```
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
```

Mount partitions
Now we can mount created partitions in preparation for installing Arch OS.

Mount root partitions first:
```
mount /dev/sda3 /mnt
```

Then create and mount boot partition
```
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```
use ‘df’ to show mounted partitions. You can extend this command further to show type of partitions: ‘df -Th’


## Download and install Arch on mounted root partition Update mirror list
Arch install comes with populated mirror list, however you can update it with the best mirrors for your location. A mirror list can be generated using this page: https://www.archlinux.org/mirrorlist

Create a backup of the original list with following command:
```
mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bk
```

Then create a new file and paste your mirrors list into it:
```
reflector --country [Country] --latest 20 --sort rate --protocol https --save /etc/pacman.d/mirrorlist
pacman -Syy
nano /etc/pacman.d/mirrorlist
```
> More info about mirrors: https://wiki.archlinux.org/index.php/Mirrors


## Install base package
Upgrade archlinux-keyring
```
pacman -Sy archlinux-keyring
```    
Now it is time to download and install the Arch OS files and some basic packages like pacman, bash, etc
> linux = latest release    
> linux-lts =  long term support release
```
pacstrap /mnt base base-devel openssh networkmanager nano git linux linux-headers grub efibootmgr
```
Create file systems table (fstab file)
You need to create an fstab file for the currently mounted partitions, so when the Arch starts it knows which partitions to mount. Otherwise you won’t be able to boot your system.

Below command will generate the fstab file including UUID of each partition and redirect it to /mnt/etc/fstab in the Arch install.
```
genfstab -L /mnt >> /mnt/etc/fstab
```

You can then check if the file was generated succesfully:
```
cat /mnt/etc/fstab
```

## Change root directory into your new Arch install
```
arch-chroot /mnt
```
Uncomment ‘en_US.UTF-8 UTF-8’ or other locale you need in:
```
nano /etc/locale.gen 
```
In my case I’m using ‘en_US.UTF-8 UTF-8’.

Then run locale-gen to generate locale.

```
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
locale-gen
```


Configure time zone
```
ln -s /usr/share/zoneinfo/[Region]/[city] /etc/localtime
hwclock --systohc --utc
```

Configure hostname

```
echo "YourHostName" > /etc/hostname
```

Configre bootloader

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub_uefi --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```



Add user and set password
```
useradd -m -G wheel -s /bin/bash YourUserName
passwd [username]
```

configure sudo

```
echo "[username] ALL=(ALL) ALL" >> /etc/sudoers
```

Configure network
```
systemctl enable NetworkManager
```

last step is to reboot your system.

```
exit
umount /mnt/boot
umount /mnt
reboot
```

injoy your new Arch System!