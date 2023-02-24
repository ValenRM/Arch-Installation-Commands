# Arch Installation Commands
 The commands displayed below are identical to those used in the [**Arch Install**](https://github.com/ValenRM/ArchInstall) repository.

# Checking EFI Variables
 We must verify that the `efivarfs` kernel module is loaded. Therefore we are going to run the following command:
 ```
 ls /sys/firmware/efi/efivars
 ```
 Ensure that you are booting your system with UEFI enabled if the directory is not found.

 # Establishing an Internet Connection
 If you are connected via Ethernet, make sure that you can ping external sevices such as **google.com**. If you are using Wi-Fi, you are going to use [**iwd**](https://wiki.archlinux.org/title/iwd).

 - Get into an iteractive prompt using `iwctl`. The interactive prompt will be displayed with a prefix of `[iwd]#`.

 | Command | Description |
 | :--- | :--- |
 | device list | List all Wi-Fi devices |
 | station ***device*** scan | Initiate a scan for networks (*No output will be displayed*) |
 | station ***device*** get-networks | List all available Wi-Fi networks |
 | station ***device*** connect SSID | Connect to a network |

 # Creating Disk Partitions
 To check the current disks and partitions, use the following command:
 ```
 fdisk -l
 ```
 We will create three partitions
 - EFI
 - Swap
 - Root

 Launch the [**fdisk**](https://wiki.archlinux.org/title/fdisk) command-line utility by entering the following command:
 ```
 fdisk /dev/<DISK_NAME>
 ```

 ## Creating EFI Partition

 | Command | Description
 | :---: | :--- |
 | g | Creates a new GPT partition table
 | n | Creates a new partition
 | 1 | Partition Number
 |  | Leave Empty
 | +550M | EFI Partition Size

 ## Creating Swap Partition

 | Command | Description
 | :---: | :--- |
 | n | Creates a new partition
 | 2 | Partition Number
 |  | Leave Empty
 | +4G | Swap Partition Size

 ## Creating Root Partition

 | Command | Description
 | :---: | :--- |
 | n | Creates a new partition
 | 3 | Partition Number
 |  | Leave Empty
 | +100G | Root Partition Size (Leave empty for the max capacity)

 ## Defining Partition Types
 We are now going to specify the EFI partition type for partition one.

 | Command | Description
 | :---: | :--- |
 | t | Change a partition type
 | 1 | Partition number
 | 1 | EFI partition type

 We also need to set partition two as a Swap partition

 | Command | Description
 | :---: | :--- |
 | t | Change a partition type
 | 2 | Partition number
 | 19 | Swap partition type

 ## Writing Changes

 | Command | Description
 | :---: | :--- |
 | w | Write changes

 ## Formatting Partitions
 We will format all three partitions.
 ### Formatting EFI Partition
 ```
 mkfs.fat -F 32 /dev/<EFI_PARTITION>
 ```

 ### Formatting Swap Partition
 ```
 mkswap /dev/<SWAP_PARTITION>

 swapon /dev/<SWAP_PARTITION>
 ```
### Formatting Root Partition
```
mkfs.ext4 /dev/<ROOT_PARTITION>
```

# Mounting the File System
 The Root partition should be mounted to the `/mnt` directory. To do so we can use this command:
 ```
 mount /dev/<ROOT_PARTITION> /mnt
 ```

 Finally, to mount the EFI partition, we first need to create the `/mnt/boot` directory using the `mkdir` command. Then, we can use the `mount` command to attach the EFI partition to this directory by specifying the partition's device file and the mount point, which is `/mnt/boot` in this case.
 ```
 mkdir /mnt/boot

 mount /dev/<EFI_PARTITION> /mnt/boot
 ```

 # Installing Linux Kernel
 By running `pacstrap` and specifying the target installation directory *(in our case, /mnt)*, we can install the necessary packages and dependencies for the kernel to function properly.
 ```
 pacstrap -K /mnt base linux linux-firmware
 ```

 # Creating File System Table
 We will generate an **fstab** file using the `genfstab` command. This will create an fstab file in the `/mnt/etc` directory based on the current mounted file systems and devices.
 ```
 genfstab -U /mnt > /mnt/etc/fstab
 ```
 You can go ahead and verify that the fstab file has been successfully generated under `/mnt/etc/fstab` by running the following command:
 ```
 cat /mnt/etc/fstab
 ```

 # Switching Enviroments
 We will switch to the **chroot** environment using the `arch-chroot` command, with the root directory set to `/mnt`.
 ```
 arch-chroot /mnt
 ```

 ## Configuring Region & City
 We will create a symbolic link between the specified time zone file located in `/usr/share/zoneinfo/<Region>/<City>` and the `/etc/localtime` file. The `-s` option specifies that the link should be symbolic, while the `-f` option forces the creation of a new link even if one already exists.
 
 ```
 ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime
 ```

 - NOTE: If you don't know your Region and City, you can run the `timedatectl list-timezones` command.

 ## Set Current System Time
 Setting the hardware clock on the system to the current system time.
 ```
 hwclock --systohc
 ```

 ## Neovim Installation
 We need to install a text editor such as **Neovim** to edit different files.
 ```
 pacman -S neovim
 ```

 ## Uncommenting Locale
 We need to open the `/etc/locale.gen` file with **Neovim** and uncomment the locale. By uncommenting the locale you use, you are enabling that specific locale for your system, which ensures that your system uses the correct language and character encoding settings.

 - NOTE: For this example, I'm going to use `en_US.UTF-8 UTF-8` as my locale

 ```
 nvim /etc/locale.gen
 ```

 After uncommenting the locale, we must run the `locale-gen` command to apply the changes.

 ```
 locale-gen
 ```

 After the locale has been generated, we need to edit the `locale.conf` file located in `/etc/locale.conf`. Here we are going to copy the exact same local we've uncommented earlier. Keep in mind that this file specifies the system-wide locale settings, which determines the language and cultural preferences used by system services and applications. For this example, I'm going to add `LANG=en_US.UTF-8` to the `locale.conf` file.

 ```
 echo "LANG=en_US.UTF-8" > /etc/locale.conf
 ```

 ## Network Configuration
 ### Hostname
 To define our hostname, we just need to type the desired name into `/etc/hostname`.

 ```
 echo "<HOSTNAME>" > /etc/hostname
 ```
 ### Loopback  Configuration
 To configure the loopback interface, we need to add the following lines to the `/etc/hosts` file:
 
 ```
 nvim /etc/hosts

    127.0.0.1	localhost
	::1			localhost
	127.0.1.1	<HOSTNAME>.localhost	<HOSTNAME>
 ```
 
 ## User Configuration
 To create a new user account (our account), we will use the `useradd` command followed by the username:
 - NOTE: The `-m` option is crucial, because we need to create a home directory for the user.
 ```
 useradd -m <USERNAME>
 ```

 After your user has been created, create a strong password and assign it to that user using:
 ```
 passwd <USERNAME>
 ```

A password should be configured for the root user.
```
passwd <ROOT_PASSWORD>
```

## User Groups & Sudo
 Add your user to `wheel`, `audio`, `video`, `optical`, and `storage` groups:
 ```
 usermod -aG wheel,audio,video,optical,storage <USERNAME>
 ```

 ## Installing Sudo
 ```
 pacman -S sudo
 ```

 ## Configuring Sudo
 `%wheel ALL=(ALL) ALL` Should be uncommented from the Sudo configuration file. By doing so, all users who have been added to the `wheel` group will be granted the ability to execute any command.
 ```
 EDITOR=nvim visudo
 ```

 ## Grub Installation & Configuration
 To install grub, we should run the following command:
 ```
 pacman -S grub efibootmgr dosfstools os-prober mtools networkmanager wpa_supplicant base-devel
 ```
 
 After grub has been successfully installed, we need to run the `grub-install` command. Under the `--bootloader-id` flag, you should enter the name you want to be displayed on your computer's boot manager.
 ```
 grub-install --target=x86_64-efi --efi-directory=<efi-directory-path> --bootloader-id=<any name you like> --recheck
 ```

 - NOTE: Add the `--removable` flag before the `--recheck` flag if you are installing Arch Linux on a removable drive, such as an USB stick.

 To generate the configuration file for GRUB, we need to use the `grub-mkconfig` command. This command scans the system and generates a new configuration file, which is saved to the file `/boot/grub/grub.cfg`.
 ```
 grub-mkconfig -o /boot/grub/grub.cfg
 ```

 ## Final Configuration
 Enable `NetworkManager` and `wpa_supplicant.service` using the `systemctl` command:
 ```
 systemctl enable NetworkManager

 systemctl enable wpa_supplicant.service
 ```

 Synchronize time using `timedatectl`:
 ```
 timedatectl set-ntp true
 ```

 ## Exiting Chroot Enviroment
 ```
 exit
 ```

 # Finishing Installation
 To finish up the installation of Arch Linux, we need to unomunt all the partitions that were previously mounted to `/mnt`. The `-R` option stands for recursive operation. 
 ```
 umount -R /mnt
 ```

 Reboot the system using the `reboot` command, or `shutdown` command. Make sure to remove the installation media before booting.